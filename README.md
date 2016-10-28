## control-m 的 批处理程序
magapp15a:/shared/opt/SCB/pos_eod/live/scripts$ less eod_task.pl
# Copyright 2006 Standard Chartered Bank
eval 'exec perl -S $0 "$@"' if $running_under_some_shell;

#############################################################################
# SUMMARY
# Job Management script for Control-M/MxG2K EOD processing
#############################################################################


#############################################################################
# Standard Perl Libraries
#############################################################################

use English;
use Carp;
use POSIX;

use FileHandle;
use File::Basename;
use File::Copy;
use File::Path;
use Getopt::Long;
use IO::Handle;
use Sys::Hostname;
use Cwd;

#############################################################################
# LOCAL INITIALISATION
#############################################################################
# Special Perl rountine is executed the moment it is completely defined
# this allows variables to be used to access Perl libraries
BEGIN
{
   my $dirname = dirname ($PROGRAM_NAME);
   my $OWD = cwd();
   chdir $dirname . "/..";
   my $BASE_DIR = cwd();
   chdir $OWD;
   $ENV{'POS_EOD_BASE'} = $BASE_DIR;
}

#############################################################################
# Non-standard Perl Libraries
#############################################################################

use lib "$ENV{'POS_EOD_BASE'}/scbperl/lib";
use lib "$ENV{'POS_EOD_BASE'}/cpan/lib";
use lib "$ENV{'POS_EOD_BASE'}/scripts";

use SCB::Profile;
use SCB::Timer;

use pos_eod_utils;
use pos_eod_checks;

# ------------------------------------------------------------------------------
# Force perl into strict mode
use strict;
use strict 'vars';

# ------------------------------------------------------------------------------
# Check the perl version
die "Must have Perl 5.00503 or greater" if $PERL_VERSION < 5.00503;

# ------------------------------------------------------------------------------
# Setup signal handlers
setSignals();

# ------------------------------------------------------------------------------
# Autoflush stdout (for debug)
STDOUT->autoflush (1);

# ------------------------------------------------------------------------------
# Default values for SNMP traps (used if trap raised before config is loaded)
my $_def_trapid  = "6 48";	
my $_def_envfile = "/var/opt/OV/share/conf/mgdparm_%hostname%";
my $_def_bindir  = "/opt/OV/bin";
my $_def_snmpexe = "snmptrap";
my $_def_sendtrap = ". /opt/bin/snmpfunctions; sendtrap";

# ------------------------------------------------------------------------------
# Global set-up

# predeclare global variables
use vars qw(%_optctl 
	    $_true $_false $_success $_failure
	    $_profile $_logfh $_tab $_locklog
	    $_prevdate $_currdate $_timer $_sdate $_stime
	    $_verbose $_debug $_ps_srvcount
	    $_ps_user $_usrfname $_usrassign
	    $_ps_server $_svrfname $_svrassign
	    $_proc_checks $_hostname $_send_traps
	    $_fail_list $_lastdate $hourtime $weekday $jobcode $max_lvl
	    %min_thresholds %max_thresholds @max_thresholds $prev_alarm_time $receivedAlarm %sevLabel);

# set-up true and false
$_false   = 0; $_true    = (!$_false);
$_success = 0; $_failure = 1;

# set-up command line options
%_optctl = ();
GetOptions (\%_optctl, "help!", "verbose!", "nodebug!", "notrap!", "nocheck!", 
		       "jobcode=s", "config=s", "prevdate=s", "currdate=s", 
		       "psuser=s", "psserver=s", "lastdate=s",
		       "pssrvcount=s");
# show usage if help requested or job code missing
usage() if	$_optctl{'help'};
usage() unless	$_optctl{'jobcode'};

# snmp on/off switch (-nosnmp option overrides config file value)
$_send_traps  = ( $_optctl{'notrap'}  ) ? $_false : $_true;
$_proc_checks = ( $_optctl{'nocheck'} ) ? $_false : $_true;

# set verbose and debug flags
$_debug   = ( $_optctl{'nodebug'} ) ? $_false : $_true; 
$_verbose = $_optctl{'verbose'}; 

# name of host running this script
$_hostname = hostname();

# load the timer object
$_timer = new SCB::Timer ( $_optctl{'jobcode'}, $_verbose );
$_stime = $_timer->stime ( "%H%M_%S" );
$_sdate = $_timer->stime ( "%a %e %h %Y" );

# locate and load the config profile
my $_cfcat   = ( $_optctl{'config'} ) ? ".$_optctl{'config'}" : "";
my $_cfname  = "$ENV{'POS_EOD_BASE'}/config/eod_main$_cfcat.cf";
$_profile = new SCB::Profile ($_cfname);

# turn off snmp traps if config says so 
# unless already turned off by cmd opt
$_send_traps = getcfg ( "ERROR_HANDLING/SEND_TRAPS" ) if $_send_traps;

# file name for logging file locks
$_locklog = "/tmp/pos_eod_flock.log";

# tab indent for log reporting lines
$_tab = 0;

# dates set to dates in date_td.txt unless specified as cmd option
$_prevdate =( $_optctl{'prevdate'} ) ? "$_optctl{'prevdate'}" : busdate("_");
$_currdate =( $_optctl{'currdate'} ) ? "$_optctl{'currdate'}" : 
							busdate("_", "current");
# This date is got from date_la.txt
$_lastdate =( $_optctl{'lastdate'} ) ? "$_optctl{'lastdate'}" : 
							busdate("_", "last");

# NB: previous trading date is taken as the business date, since the front 
# office date will have rolled and we are processing for the day just gone

# murex user for runnning processing script
# allocated at runtime by assignUser function, unless specified as cmd option
$_usrassign = $_optctl{'psuser'};
$_ps_user = "";

# murex server on which processing script should be run
# allocated at runtime by assignServer function, unless specified as cmd option
$_svrassign = $_optctl{'psserver'};
$_ps_server = "";

# count of sessions to be allocated from the server file (defaults to 1)
if (defined $_optctl{'pssrvcount'}) { 
   $_ps_srvcount = $_optctl{'pssrvcount'};
   report "Server sessions to allocate is $_ps_srvcount";
} else {
    $_ps_srvcount = 1;
}
#$_ps_srvcount = ( $_optctl{'pssrvcount'} ) ? "$_optctl{'pssrvcount'}" : 1;

# filename holding user and server allocation tables respectively
$_usrfname;
$_svrfname;

# Holds the list of failures of checks to be reported at the end of
# the sysout at the end of main().
$_fail_list = "";


$max_lvl = 256;
# ------------------------------------------------------------------------------
# Call the main routine
exit main();

# ------------------------------------------------------------------------------
# Display the usage panel
sub usage() {
    print "$PROGRAM_NAME\n",
    
    "\t-jobcode=<jobcode>        - job code to run\n",
    "\t[-config=<category>]      - non-default config category (e.g. dev)\n",
    "\t[-prevdate=<date>]        - previous trading date in YYYY_MM_DD format\n",
    "\t[-currdate=<date>]        - current trading date in YYYY_MM_DD format\n",
    "\t[-lastdate=<date>]        - last trading date in YYYY_MM_DD format\n",
    "\t[-psuser=<mxguser>]       - mxg user for processing script (override)\n",
    "\t[-psserver=<server alias>]- server alias for proc script (override)\n",
    "\t[-pssrvcount=<number>]    - # of server sessions to allocate (def 1)\n",
    "\t[-notrap]                 - suppress snmp traps\n",
    "\t[-nocheck]                - suppress post-processing checks\n",
    "\t[-verbose]                - show extra debug information\n",
    "\t[-nodebug]                - prevent logs going to standard out\n",
    "\t[-help]                   - show this help file\n";
    
    exit 0;
}

#############################################################################
sub setAlarm($$$) {
   my($signal, $period, $alarmpid) = @_;
   my($pid);

   if ($pid = fork()) {
      verbose "setAlarm $signal, $period, $alarmpid";
      return;
   } elsif (defined $pid) {
      # child pid
      sleep $period;
      kill $signal, $alarmpid;
      #verbose "$$ send $signal to $alarmpid";
      exit 0;
   } else {
      die "Failed to fork: $!\n";
   }
}

#############################################################################
sub setNewAlarm() {
   if ($#max_thresholds >= 0) {
      my($time) = pop @max_thresholds;
      my($alarm_time) = $time - $prev_alarm_time;
      $prev_alarm_time = $time;
      setAlarm("USR1", $alarm_time, $$);
   }
}

#############################################################################
sub alarmHandler {
   my($signame) = @_;

   $_verbose = $_false;
   $receivedAlarm = $prev_alarm_time;
   verbose "****Received signal SIG$signame $max_thresholds{$receivedAlarm}";
   setNewAlarm();


    # test if over-run failures
       my($dlvl) = $max_thresholds{$receivedAlarm};
       my($dthreshold) = sprintf("%d", $receivedAlarm);
       if ($max_lvl > $dlvl) { $max_lvl = $dlvl; }
       my($trap_id, $stop, $email) = getErrConfig ( $dlvl );
       $_fail_list .= "Job completion exceeded time threshold ($dthreshold secs)\n";
       report "Job completion exceeded time threshold ($dthreshold secs)";
       report "    Level=$dlvl trap=$trap_id stop=$stop email=$email";
       #$error_count_crt++ if ($stop eq "YES");
       #$error_count++;
       ($trap_id, $stop, $email) = getErrConfig ( $max_lvl );


        my($snmptxt)  =
        "\"job $jobcode (started on $_sdate @ $_stime) failed a level" .
        " $max_lvl check. Job completion exceeded time threshold ($dthreshold secs).\"";
           
        if ( $email eq "YES" ) { send_email($snmptxt); }

        if ( $_send_traps and ( $trap_id ne "NONE" ) ) {
           report "raising SNMP trap";
           snmpTrap ( $trap_id, $snmptxt ) if $_send_traps;
           report "raised SNMP trap (ok)";
        }

        #if ($stop eq "YES") { 
        #   report "ABORT Execution";
        #   exit 1; 
        #}

}

# ------------------------------------------------------------------------------
# Set-up signal handler
sub setSignals() {
    $SIG{INT}     = \&sigTerm;
    $SIG{TERM}    = \&sigTerm;
    $SIG{__DIE__} = \&exitDie;

}

# ------------------------------------------------------------------------------
# Signal handler
sub sigTerm($) {
    my $sig = shift; chomp $sig;

    my $date = timestamp ("%a %e %h %Y");
    my $time = timestamp ("%T");

    $_tab = 0;
    
    report (50);
    report "received signal ($sig), will attempt de-allocation if required";

    # deallocate sessions from processig script users and servers
    deallocAll();

    report (50); 
    report "exiting after receiving signal ($sig) on $date @ $time";

    exit 1;
}

# ------------------------------------------------------------------------------
# Exit (die) handler
sub exitDie($) {
    my $err = join ('', @_); chomp $err;

    my $date = timestamp ("%a %e %h %Y");
    my $tim  = timestamp ("%T");

    $_tab = 0;
    report "FATAL ERROR: $err";
    report (50); report "terminated with error on $date @ $tim";
    
    # de-allocate sessions from processig script users and servers
    deallocAll();

    if ( $_send_traps ) {

	report "raising SNMP trap";
	
	my $jobcode = $_optctl{'jobcode'};
	my $trap_id = "";
	my $snmptxt =
	
	"\"job $jobcode (started on $_sdate @ $_stime) " .
	"had a FATAL error, logfile may be available as [ " .
	"eod_job.$_stime.$jobcode.log ] err [ $err ]\"";
	
	#snmpTrap ( $trap_id, $snmptxt );
	
	#report "raised SNMP trap (ok)";
    }

    carp ($err);
}

# ------------------------------------------------------------------------------
# Attempt de-allocation of assigned servers and users on unexpected exit
sub deallocAll {

    if ( $_ps_user ne "" ) {

	# de-allocate user
	report "WARNING: user $_ps_user not deallocated" 
	    unless userAssign ( $_usrfname, $_ps_user );
    }

    if ( $_ps_server ne "" ) {
    
	# de-allocate server
	report "WARNING: server $_ps_server not deallocated" 
	    unless serverAssign ( $_svrfname, $_ps_srvcount, $_ps_server );
    } 
}

# ------------------------------------------------------------------------------
# Write contents of config file to standard out
sub reportConfig(;$){
    my $basepath = shift;
    $basepath = "/" unless defined $basepath;
    
    report "profile contents for $_cfname (from path $basepath) :";
    
    my $key;
    foreach $key ( sort keys %{$_profile->{'config'}} ) {

	next unless $key =~ /^$basepath(.*)$/;
	report "\t$key == " . clean ( $_profile->get_value ( $key ) );
    }
    report "end of profile contents for $_cfname (from path $basepath)";
}

#############################################################################
# set min and max time thresholds (if any) from configuration
sub setTimeThresholds($) {
   my($jobcode_lc) = @_;

   %min_thresholds = ();
   %max_thresholds = ();
   @max_thresholds = ();
   undef $receivedAlarm;

   my($value) = getcfg( "THRESHOLDS/JOBS/$jobcode_lc" );
   if ($value eq "") { return; }

   $value =~ s/\\//g;
   while ($value =~ m/([<>])\s*([\d:]+\s*,\w+)\s*(.*)/) {
      my($threshType) = $1;
      my($threshold) = $2;
      my($time, $sev);
      $value = $3;
      $threshold =~ s/\s//g; #remove all spaces
      if (($time, $sev) = $threshold =~ m/([\d:]+),(\w+)/) {
         if ($sev !~ m/^\d+$/) {
            if ( defined $sevLabel{$sev} ) {
               $sev = $sevLabel{$sev};
            } else {
               die "Failed to recognise Severity label ($sev)";
            }
         }
         if ($time =~ m/(\d*):(\d*)$/) {
            my($hrs) = 0;
            my($mins) = 0;
            my($secs) = 0;
            $secs = $2 if ( defined $2 );
            $mins = $1 * 60 if ( defined $1 );
            if ($time =~ m/(\d+):\d*:\d*$/) {
               $hrs = $1 * 3600;
            }
            $time = $hrs + $mins + $secs;
         }
         if ($time > 0) {
            $time = sprintf("%08d", $time);
            if ($threshType =~ /</ ) {
               $min_thresholds{$time} = $sev;
            } else {
               $max_thresholds{$time} = $sev;
            }
         }
      }
   }

   my($time);
   foreach $time (reverse sort keys %min_thresholds) {
      verbose "min threshold at $time is $min_thresholds{$time}";
   }
   foreach $time (reverse sort keys %max_thresholds) {
      verbose "max threshold at $time is $max_thresholds{$time}";
      push @max_thresholds, $time;
   }
}

#############################################################################
sub setSevLabels() {
    my $bkey    = "ERROR_HANDLING/POSTPROC_HANDLING";

    %sevLabel = ();
    # determine the severity level
    my $severity = "DEF";
    my $sevlist  = getKeys ( "$bkey/SEVERITY" );

    foreach ( @$sevlist ) {
        my $severitydef = getcfg ( "$bkey/SEVERITY/$_" );
        my ($min, $max) = split /,/, $severitydef;
        $sevLabel{$_} = $min;
        verbose "sevLabel{$_} = $min";
    }
}

#############################################################################
sub earlyFinish() {

   my($duration) = getduration();
   verbose "duration = $duration secs";
   my($time);
   foreach $time (reverse sort keys %min_thresholds) {
      #verbose "min threshold at $time is $min_thresholds{$time}";
      if ($duration < $time) {
         verbose "RAISE $min_thresholds{$time}";
         return $time;
      }
   }
   return 0;
}

#############################################################################
# ------------------------------------------------------------------------------
# Main body
sub main() {
    $jobcode = $_optctl{'jobcode'};
    my $jobcode_lc = lc ( $jobcode );
    my $jobsdir = getcfg ( "JOB_DEFS_DIR" );

    my $logdir = getcfg ( "LOG_DIR" ) . "/$_prevdate";
    my $logname = "$logdir/eod_job.$_stime.$jobcode.log";
    
    # make the log dir if it not already present, loop in case we
    # conflict with another instance
    my $maxdcnt = (getcfg ( "LOG_DIR_CREATE_TRIES" ) or 5);
    my $dcnt = 0;
    until ( -d $logdir) {
       ( $dcnt++ <= $maxdcnt ) or die "couldn't create dir $logdir: $!";
       mkdir ( $logdir , 0775 ) and next;
       sleep 1;
    }

    # make sure directory has rw permission on group
    my $count = chmod 0775, $logdir;
    print "WARNING: could not chmod (0775) $logdir" if ($count != 1);
    
    die "logfile $logname already exists" if -e $logname;

    # open the logfile
    $_logfh = new IO::File ($logname, "w+") 
	or die "unable to open logfile $logname: $!\n";

    # load up job definition
    report "Loading job definition $jobsdir/$jobcode_lc.cf";
    $_profile->interpolate ( "$jobsdir/$jobcode_lc.cf" );

    report "running job $jobcode ($_sdate @ $_stime) on $_hostname (pid=$PID)";
    report "business processing date is $_prevdate";
    report "SNMP traps are " . ( ( $_send_traps ) ? "ON" : "OFF" );
    report "post-processing checks are " . ( ( $_proc_checks ) ? "ON" : "OFF" );
    
    # set time thresholds (if any)
    $SIG{USR1} = $SIG{USR2} = \&alarmHandler;
    setSevLabels();
    setTimeThresholds($jobcode_lc);
    $prev_alarm_time = 0;
    setNewAlarm();

    reportConfig() if $_verbose;
    
    report "running job commands...";

    $_tab++;
    
    my $retcode = runJob ( $jobcode ); 

    $_tab--; 
    
    report "end of job commands (last return code = $retcode)";

    my $status = $_success;

    if ( $_proc_checks ) {
    
	report "running post-processing checks for $jobcode"; 
    
	$_tab++;
    
	# do post-processing checks
	$status = postProcCheck ( $jobcode, $retcode ) if $_proc_checks; 
    
	$_tab--;

	report "end of post-processing checks for $jobcode (" .
		( ( $status ) ? "failed)" : "ok)" );
    }
    else {

	report "post-processing checks are off, returning $_success by default";
    }

    # List the checks that failed at the end of the sysout
    if ( $_fail_list ) {
	report $_fail_list;
    }

    report "end of job $jobcode (". ( ( $status ) ? "failed)" : "ok)");
    report "job $jobcode took " . ( $_timer->duration() );
    
    return $status;
}

# ------------------------------------------------------------------------------
# Run job for specified job code
sub runJob($;$) {
    my $jobcode = shift;
    my $jobcode_lc = lc ( $jobcode );

    my $jobsdir = getcfg ( "JOB_DEFS_DIR" );
    
    my $jobtype = getcfg ( "/$jobcode/JOB_TYPE" );
    my $jobname = getcfg ( "/$jobcode/JOB_NAME" );

    my $retcode;
    my($startTime) = time();
    if ( $jobtype eq "PROCESSING_SCRIPT" ) {

	$retcode = runProcScript ( $jobname );
    }
    elsif ( $jobtype eq "INTERFACE_SCRIPT" ) {

	$retcode = runInterfaceScript ( $jobname );
    }
    else { 
    
	die "unknown job type \"$jobtype\" for job code $jobcode";
    }
    
    
    
    return $retcode;
}

# ------------------------------------------------------------------------------
# Check the status of the previous executed command
sub postProcCheck($$) {
    my ($jobcode, $retcode) = @_;
    my ($trap_id, $stop, $snmptxt);
    # Get the SNMP Trap info for Sev 3 for the information alerts
    my ($sev3_trap_id, $sev3_stop, $email) = getErrConfig ( 32 );
    my $error_count_crt = 0;
    my $error_count = 0;
    my $email = "NO";

    my $bkey = "/$jobcode/POST_PROC_CHECKS";
    my $checklist = getKeys($bkey);
    my $check;

    foreach $check ( @$checklist ) {
	my $checkdef =  getcfg ( "$bkey/$check" );
	my ($lvl, $type, @args) = split /\|/, $checkdef, -1;

	substGenPlaceholders ( @args );
	substPsPlaceholders  ( @args );

	my $errormsg = pop @args;
	
	# for return code check prepend arg list with the job return code
	unshift @args, $retcode if $type eq "returnCode";

	my $argstr; $argstr .= "'$_' " foreach ( @args );
	
	my $feedback = [];

	# all check functions take ref to feedback list as first arg
	unshift @args, $feedback;
	
	report "running level $lvl check: $check $type ( $argstr )";

	$_tab++;
	
	# create a reference to a function as named by $type variable
	# value for $type is taken from the post-proc check config line
	my $check_func = \&$type; 

	# we'll handle errors ourselves after check function returns
	$SIG{__DIE__} = \&return;

	# call the eod check function
	my $ok = eval { &$check_func ( @args ) };

	# reset signal handlers
	setSignals();

	# handle die signals from eval
	if ( $EVAL_ERROR ) {
	    die $@ unless ( $@ =~ /^Undefined subroutine.*$type called/ );
	    report "\tunknown subroutine ($type), forcing fail";
	    $errormsg = "no subroutine $type exported from package pos_eod_checks.pm";
	    $lvl = 63; # Raise this as an information trap
	    $ok = $_false;
	}


	unless ( $ok ) {
	    # Report the error to the log
	    report "ERR: $errormsg";

	    ($trap_id, $stop, $email) = getErrConfig ( $lvl );
	    # Keep track of the worst severity error
	    $max_lvl = $lvl if ( $max_lvl > $lvl );

	    # Count the number of errors for the final alert
	    $error_count_crt++ if ($stop eq "YES");
	    $error_count++;

	    # Set up error message for SNMP Trap and sysout
	    my $fbtext; $fbtext .= "$_ " foreach ( @$feedback );

	    $snmptxt  = 
	    "\"job $jobcode (started on $_sdate @ $_stime) failed level " .
	    "$lvl check [ $check $type ( $argstr ) ] see logfile [ "      .
	    "eod_job.$_stime.$jobcode.log ] err [ $errormsg ] feedback [" .
	    " $fbtext]\"";

	    $_fail_list .= "--- " . $error_count . " -----------------------\n";
	    $_fail_list .= $snmptxt . "\n";

	    # Raise an SNMP Trap if required
	    if ( $_send_traps and ( $trap_id ne "NONE" ) ) {

		report "raising SNMP trap";

		# Raise SNMP Trap at level 3 for information
		# Job error trap raised later will give the max severity
		#snmpTrap ( $sev3_trap_id, $snmptxt );
		
		#report "raised SNMP trap (ok)";
	    }
	    else {
		report "no SNMP rasied";
	    }
	}  

	$_tab--;
	report "end of check (" . ( ($ok) ? "ok)" : "failed)" );

    }
 
    # test if over-run failures
    if (defined $receivedAlarm) {
       my($exceedlvl) = $max_thresholds{$receivedAlarm};
       my($dthreshold) = sprintf("%d", $receivedAlarm);
       if ($max_lvl > $exceedlvl) { $max_lvl = $exceedlvl; }
	   ($trap_id, $stop, $email) = getErrConfig ( $exceedlvl );
       $_fail_list .= "Job completion exceeded time threshold ($dthreshold secs)\n";
       report "Job completion exceeded time threshold ($dthreshold secs)";
       report "    Level=$exceedlvl trap=$trap_id stop=$stop email=$email";
       $error_count_crt++ if ($stop eq "YES");
       $error_count++;
    }

    # test if completed too quickly
    my($earlyrc) = earlyFinish();
    if ( $earlyrc ) {
       my($dlvl) = sprintf( "%d", $min_thresholds{$earlyrc});
       if ($max_lvl > $dlvl) { $max_lvl = $dlvl; }
       my($dthreshold) = sprintf("%d", $earlyrc);
	   ($trap_id, $stop, $email) = getErrConfig ( $dlvl );
       $_fail_list .= "Job completed prior to time threshold ($dthreshold secs)\n";
       report "Job completed prior to time threshold ($dthreshold secs)";
       report "    Level=$dlvl trap=$trap_id stop=$stop email=$email";
       $error_count_crt++ if ($stop eq "YES");
       $error_count++;
    }


    # Did we have an error?
    if ( $_fail_list ) {

        # Put totals at end of failure list
        $_fail_list .= "Error Counts -----------------------------------\n";
	    $_fail_list .= "Critical: " . $error_count_crt . "\n";
        $_fail_list .= "All:      " . $error_count . "\n";



	# Find the config of the most severe error we got
        ($trap_id, $stop, $email) = getErrConfig ( $max_lvl );

	if ( $_send_traps and ( $trap_id ne "NONE" ) ) {

	    report "raising SNMP trap";

	    $snmptxt  = 

	    "\"job $jobcode (started on $_sdate @ $_stime) failed a level" .
	    " $max_lvl check. There were $error_count_crt critical " .
	    "out of $error_count total errors. Please see the sysout for " .
	    "details.\"";
			   
	    snmpTrap ( $trap_id, $snmptxt ) if $_send_traps;
		
	    report "raised SNMP trap (ok)";
	}
	else {
	    report "no SNMP rasied";
	}
	
    if ( $email eq "YES" ) { send_email($snmptxt); }

	# If the most severe error was critical then return a failure
	# for the job
	return $_failure if ( $stop eq "YES" );

    }
	
    # no critical checks failed so return success
    return $_success;
}

    
# ------------------------------------------------------------------------------
# Substitute general placeholders with real values
sub substGenPlaceholders(@) {
    my $jobcode = $_optctl{'jobcode'};
    my $logdir = getcfg ( "LOG_DIR" ) . "/$_prevdate";
    my $mxgdir = getcfg ( "MXG_DIR" );
    my $time = timestamp ("%T");
    
    my $curr_cymd = substr ( $_currdate, 0, 4 ) . 
		    substr ( $_currdate, 5, 2 ) . 
		    substr ( $_currdate, 8, 2 );

    my $curr_dmy  = substr ( $_currdate, 8, 2 ) . 
		    substr ( $_currdate, 5, 2 ) . 
		    substr ( $_currdate, 2, 2 );

    my $prev_cymd = substr ( $_prevdate, 0, 4 ) . 
		    substr ( $_prevdate, 5, 2 ) . 
		    substr ( $_prevdate, 8, 2 );

    my $prev_dmy  = substr ( $_prevdate, 8, 2 ) . 
		    substr ( $_prevdate, 5, 2 ) . 
		    substr ( $_prevdate, 2, 2 );

    my $last_cymd = substr ( $_lastdate, 0, 4 ) . 
		    substr ( $_lastdate, 5, 2 ) . 
		    substr ( $_lastdate, 8, 2 );

    my $last_dmy  = substr ( $_lastdate, 8, 2 ) . 
		    substr ( $_lastdate, 5, 2 ) . 
		    substr ( $_lastdate, 2, 2 );

    # search and replace place holders
    foreach ( @_ ) {

	# time and date placeholders
        s/%prevdate%/$_prevdate/g;
        s/%currdate%/$_currdate/g;
        s/%lastdate%/$_lastdate/g;
	
	s/%currdate_ccyymmdd%/$curr_cymd/g;
	s/%prevdate_ccyymmdd%/$prev_cymd/g;
	s/%lastdate_ccyymmdd%/$last_cymd/g;
	s/%currdate_ddmmyy%/$curr_dmy/g;
	s/%prevdate_ddmmyy%/$prev_dmy/g;
	s/%lastdate_ddmmyy%/$last_dmy/g;
	 
	s/%time%/$time/g;
	s/%stime%/$_stime/g;
	
	# mxg directory placeholders
        s/%mxgdir%/$mxgdir/g;
        s/%preportdir%/$mxgdir\/scb\/reports\/$_prevdate/g;
        s/%creportdir%/$mxgdir\/scb\/reports\/$_currdate/g;
        s/%lreportdir%/$mxgdir\/scb\/reports\/$_lastdate/g;
	    
	# log directory placeholders
        s/%logfile%/$logdir\/eod_job.$_stime.$jobcode.log/g;	

	# miscellaneous
	s/%userid%/$REAL_USER_ID/g;
	s/%jobcode%/$jobcode/g;
	s/%retcode%/\$retcode/g;
    }
}

# ------------------------------------------------------------------------------
# Substitute processing script specific placeholders with real values
sub substPsPlaceholders() {
    my $jobcode = $_optctl{'jobcode'};
    my $workdir = getcfg ( "WORK_DIR" ) . "/$_prevdate";

    # search and replace place holders
    foreach ( @_ ) {
    
	# processing script placeholders
	s/%xmlanswer%/%psrundir%\/answer.xml/g;
	s/%xmllog%/%psrundir%\/log.xml/g;
	s/%xmlheader%/%psrundir%\/header.xml/g;
	s/%xmlrequest%/%psrundir%\/xml_request.xml/g;
	s/%psrundir%/$workdir\/eod_job.$_stime.$jobcode.psr/g;
    }
}

# ------------------------------------------------------------------------------
# Run a processing script
sub runProcScript($) {
    my ($procScript) = @_;

    # first see if we can allocate a session to a user
    $_svrfname = getcfg ( "PROC_SCRIPT_ENV/SERVER_ALLOC_FILE"   );
    $_usrfname = getcfg ( "PROC_SCRIPT_ENV/USER_ALLOC_FILE" );

    # assign a user to run the proc script 
    die "ERROR: couldn't allocate user for proc script $procScript" 
        unless userAssign ( $_usrfname, $_ps_user, $_usrassign );
    # assign a server on which to run the proc script
    die "ERROR: couldn't allocate server for proc script $procScript" 
	unless serverAssign ( $_svrfname, $_ps_srvcount, $_ps_server,
	$_svrassign );

    my $cdir = getcfg ( "CLIENT_DIR" );
    my $java = getcfg ( "PROC_SCRIPT_ENV/XML_REQUEST/JAVABIN" ) . "/java";

    die "mx client dir not in config key /SCB/POS_EOD/CLIENT" if ($cdir eq "");

    my $rundir = buildXmlFiles ( $procScript, $_ps_user );

    # construct the command and parameters
    my $cmd = "cd $cdir ; $java";
    my $args = buildPsCmdArgs ( "$rundir/xml_request.xml" );
    
    # always report output for main job command
    my $old_verbose = $_verbose; $_verbose = $_true;

    # execute command
    my $retcode = execute ( $cmd, $args );
    
    $_verbose = $old_verbose;
 
    # de-allocate sessions from user and server
    deallocAll();
    
    return $retcode;
}

# ------------------------------------------------------------------------------
# Run an interface script
sub runInterfaceScript($;$) {
    my $ifscript = shift;
    my $retcode = $_failure;
    
    my $jobcode = $_optctl{'jobcode'};
    my $jobcode_lc = lc ( $jobcode );
    
    my $mxdir = getcfg ( "MXG_DIR" );
    my $jddir = getcfg ( "JOB_DEFS_DIR" );
    
    reportConfig ( "/$jobcode" ) if $_verbose;

    die "config for $jobcode not found (key name may be wrong in file " .
	"$jddir/$jobcode_lc.cf)" unless ( @{getKeys("/$jobcode")} > 0 );

    # load up environment variables
    loadInterfaceEnv();
    
    # check required files are present
    my $filekeys = getKeys ( "/$jobcode/REQUIRED_FILES" );
    
    if ( @$filekeys ) {
    
	report "checking for required files";

	foreach ( @$filekeys ) {

	    my $reqdfile = getcfg ( "/$jobcode/REQUIRED_FILES/$_" );

	    die "cannot run $jobcode job, required file missing: $reqdfile"
		unless  -r "$reqdfile";
	
	    verbose "\tfound file $reqdfile (ok)";
	}
	
	report "finished checking for required files";
	report "converting required files to UNIX format";
    
	# convert files to unix format (mxg reports produce dos format)
	foreach ( @$filekeys ) { 
    
	    my $reqdfile = getcfg ( "/$jobcode/REQUIRED_FILES/$_" );
	
	    verbose "\tconverting file $reqdfile";
	    
	    ( dos2unix $reqdfile == $_success ) 
		or die "couldn't convert $reqdfile to unix format";
	}
      
	report "finished converting required files to UNIX format";
    }

    # run each of the specified commands
    my $cmdkeys = getKeys ( "/$jobcode/COMMANDS" );
    foreach ( sort @$cmdkeys ) {

	my $command = getcfg ( "/$jobcode/COMMANDS/$_" );
	
	my ($cmd, $args) = cmdparse ( $command );
	
	substGenPlaceholders ( $cmd );
	substGenPlaceholders ( @$args );

	# always report output for main job command
	my $old_verbose = $_verbose; $_verbose = $_true;
	
	# execute command 
	$retcode = execute ( $cmd, $args );

	$_verbose = $old_verbose;

	# bail out if command fails, i.e. don't process any more commands
	last if ( $retcode ne $_success );
    }

    return $retcode;
}

# ------------------------------------------------------------------------------
# load up the environment variable values needed by the interface scripts
sub loadInterfaceEnv() {
    my $keys = getKeys ( "INTERFACE_ENV" );
    my $var;
    my $depth = 0;

    my @exclusions = ( "POS_EOD_BASE", "SYBASE", "SYBASE_OCS", "CCLPROFILE.*", 
		       "LD_LIBRARY_PATH", "PATH" );

    # need to set blank values for paths if not defined externally
    # prevents infinite looping when trying to substitute in self-references
    $ENV{'LD_LIBRARY_PATH'} = "" if ( !defined $ENV{'LD_LIBRARYPATH'} );
    $ENV{'PATH'}	    = "" if ( !defined $ENV{'PATH'}	      );
    
    # ensure we only use environment as defined in config file by clearing all
    # env vars, but don't clear env vars listed in exclusions above
    foreach $var ( keys %ENV ) {

	next if ( ( grep /^$var$/ , @exclusions ) > 0 );
	delete $ENV{$var};
    }

    # read config key values into env vars without substituting nested env vars
    # (ensures all values are available for loop below which does substitute)
    foreach $var ( @$keys ) {
	    
	next if defined $ENV{$var};
	$ENV{$var} = getcfg ( "INTERFACE_ENV/$var", $_true );
    }

    # remove PATH and LD_LIBRARY_PATH to allow appending to existing value
    pop @exclusions; pop @exclusions;
    
    # read config key values again this time do substitution for nested env vars
    # also substitute in values for general %% placeholders
    foreach $var ( @$keys ) {

	next if ( ( grep /^$var$/ , @exclusions ) > 0 );
	$ENV{$var} = getcfg ( "INTERFACE_ENV/$var" );
	substGenPlaceholders ( $ENV{$var} );
    }
    
    verbose "environment variables are:";

    if ( $_verbose ) {
	foreach ( sort keys %ENV ) { report "\tenv variable $_==$ENV{$_}"; }
    }

    verbose "end of environment variable list";
    
    return 1;
}

# ------------------------------------------------------------------------------
# Build the parameters to pass to the PS command
sub buildPsCmdArgs($) {
    my $xmlfile = shift;
    my @args;
    my $bkey = "PROC_SCRIPT_ENV/XML_REQUEST";
    my $jobcode = $_optctl{'jobcode'};
    
    my $fs_host  = getcfg ( "$bkey/MXJ_FILESERVER_HOST" );
    my $fs_port  = getcfg ( "$bkey/MXJ_FILESERVER_PORT" );
    my $jar_list = getcfg ( "$bkey/MXJ_JAR_FILELIST"    );
    my $jobjavaparms = getcfg ( "/$jobcode/JAVA_PARM_LIST" );
    my $nicknameext = getcfg ( "/$jobcode/NICKNAME_EXTENSION" );
    $nicknameext = "" if $nicknameext eq "null";

    push @args, 
	"-cp " . 
	getcfg ( "$bkey/MXJ_BOOT" );
	
    push @args,
	"-Djava.security.policy=" . 
	getcfg ( "$bkey/MXJ_POLICY" );
	
    push @args,
	"-Djava.rmi.server.codebase=http://" .
	"$fs_host:$fs_port/$jar_list murex.rmi.loader.RmiLoader";
	
    push @args,
	"/MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript";
	
   #MXJ HOST and PORT no longer required for 2.10 
       #push @args,
       #"/MXJ_HOST:" . 
       #getcfg ( "$bkey/MXJ_HOST" );
	
    #push @args,
    #"/MXJ_PORT:" . 
    #getcfg ( "$bkey/MXJ_PORT" );

    #MXJ SITE NAME required for 2.10
    push @args,
        "/MXJ_SITE_NAME:" .
        getcfg ( "$bkey/MXJ_SITE_NAME" );
	
    push @args,
	"/MXJ_PLATFORM_NAME:" . 
	getcfg ( "$bkey/MXJ_PLATFORM_NAME" );
	
    push @args,
	"/MXJ_PROCESS_NICK_NAME:" . 
	getcfg ( "$bkey/MXJ_PROCESS_NICK_NAME" ) .
	"$_ps_server\_$_ps_user$nicknameext";
 	
    push @args,
	"/MXJ_CONFIG_FILE:" . $xmlfile;

    # Optional parms provided in the job's config file in jobdefs
    # For some reason a config item which is blank gets returned
    # as the string "null" by getcfg.
    if ( $jobjavaparms and $jobjavaparms ne "null" ) {
        push @args, $jobjavaparms;
    }

    return \@args;
}

# ------------------------------------------------------------------------------
# Build XML run files for processing scripts service
sub buildXmlFiles($$) {
    my ($pscript , $user) = @_;
    my $jobcode = $_optctl{'jobcode'};

    my $rname = "eod_job.$_stime.$jobcode.psr";
    
    my $templateDir = getcfg ( "XML_DIR" );
    my $runtimeDir  = getcfg ( "WORK_DIR") . "/$_prevdate/$rname";
    
    mkpath ("$runtimeDir", 0, 0777);

    buildXmlReqFile ( $templateDir, $runtimeDir, $user );
    buildXmlQryFile ( $pscript, $templateDir, $runtimeDir );

    return $runtimeDir;
}

# ------------------------------------------------------------------------------
# Build XML request file
sub buildXmlReqFile($$$) {
    my ($tdir, $rdir, $user) = @_;
    
    my $ifh = new IO::File ("$tdir/xml_request.xml", "r")
	or die "unable to open $tdir/xml_request.xml template file: $!";

    my $ofh = new IO::File ("$rdir/xml_request.xml", "w")
	or die "unable to open $rdir/xml_request.xml for writing: $!";
  
    verbose ("xml request file:");
    
    my $service = getcfg( "PROC_SCRIPT_ENV/XML_REQUEST/MXJ_PROCESS_NICK_NAME" );
    my $jobcode = $_optctl{'jobcode'};
    my $nicknameext = getcfg ( "/$jobcode/NICKNAME_EXTENSION" );
    $nicknameext = "" if $nicknameext eq "null";
    
    while ( <$ifh> ) {
	chomp;

	s/%service_nickname%/$service$_ps_server\_$_ps_user$nicknameext/g;
	s/%xml_request_header%/$rdir\/header.xml/g;
	s/%xml_request_body%/$tdir\/ps_body.xml/g;
	s/%xml_request_answer%/$rdir\/answer.xml/g;
	s/%xml_request_log%/$rdir\/log.xml/g;

	print $ofh "$_\n";
	verbose ("\t$_");
    }

    verbose ("end of xml request file");
}

# ------------------------------------------------------------------------------
# Build XML query (header) file, called recursively for each additioanl ps item 
sub buildXmlQryFile($$$;$$$) {
    my ($pscript, $tdir, $rdir, $isitem, $itemidx, $ofh) = @_;
    my $jobcode = $_optctl{'jobcode'};
    
    my $ifname = $isitem ? "$tdir/ps_item.xml" : "$tdir/ps_query.xml";
    
    my $ifh = new IO::File ( $ifname, "r" ) 
		or die "unable to open $ifname template file: $!";
    
    unless ( $isitem ) {
	$ofh = new IO::File ("$rdir/header.xml", "w")
		or die "unable to open $rdir/header.xml for writing: $!";
	
	my $jddir = getcfg ( "JOB_DEFS_DIR" );
	
        reportConfig ( "/$jobcode" ) if $_verbose;
	verbose ( "xml query header file:" );
    }
    
    while ( <$ifh> ) {
	chomp;
	if ( $_ ne "%script_items%" ) {
	    
	    # substitute %% tokens with values specified in config

	    # special case for script name 
	    $_ =~ s/\%psname\%/$pscript/g;

	    # now substitute any remaining %% tokens on this line
	    $_ = substPsCfg ( $_, $isitem ?  "/$jobcode/ITEM$itemidx" : 
					     "/$jobcode/HEADER" );
	    print $ofh "$_\n";
	    verbose "\t$_";
	}
	else {
	    
	    my $idx = 1;
	    while ( getcfg ( "/$jobcode/ITEM$idx/NAME" ) ) {
		
		buildXmlQryFile ($pscript, $tdir, $rdir, $_true, $idx, $ofh);
		$idx++;
	    }
	}
    }
    verbose ("end of xml query header file") unless $isitem;
}

# ------------------------------------------------------------------------------
# Allocate or de-allocate a physical server from the server allocation file
sub serverAssign($$;$$) {
    my ($fname, $count, $server, $serverwanted) = @_;
    my $oldserver = $server;
    my $row;
    
    # set getserver flag according to operation (allocate or de-allocate)
    # allocate (no server name supplied) or de-allocate (server name supplied)
    my $getserver = ( $server eq "" );

    if ((!$getserver) && ($count <= 0)) { return $_true; } # nothing to deallocate

    report "trying to ". ( ($getserver) ? "" : "de-" ) . "allocate server " . 
	   "$server";

    # open server allocation file
    my $fh = new IO::File ( "$fname", "r+" )
	or die "couldn't open file $fname ($!)";

    $fh->autoflush( 1 );

    verbose "trying to get write-lock on server allocation file: $fname ($count)";
    
    my $locklog = "$_locklog.server";
    
    # try to get exclusive lock on server allocation file
    unless ( lockfile ( $fh ) ) {
	report "couldn't lock server allocation file: $fname";
	showlock( $locklog );
	die "couldn't lock server allocation file: $fname";
    }

    loglock ( $locklog, $fname );

    verbose "write-locked server allocation file (ok)";

    # read file into server table hash
    my $servertable = readAllocFile ( $fh, $fname );
   
    # try to allocate session against specified server
    # If we are allocating more than one session, allocate them to the
    # same server entry
    if ( $getserver ) {
        my $minsessions = -1;
        my $lowest = -1;
        for ( my $idx = 0; $idx <= $#{$servertable}; $idx++ ) {

            # Get the current row
            my $row = $servertable->[$idx];

            # Loop if the server is disabled
            verbose "$row->[0] disabled." if ( $row->[2] eq "disabled" );
            next if ( $row->[2] eq "disabled" );
            # Loop if a specific server is wanted and this isn't it
            verbose "$row->[0] is not reqd server $serverwanted."
                if ( $serverwanted and $row->[0] ne $serverwanted );
            next if ( $serverwanted and $row->[0] ne $serverwanted );

            if ( $minsessions == -1 or $row->[1] < $minsessions ) {
                $minsessions = $row->[1];
                $lowest = $idx;
            } 
        }
        if ( $lowest < 0 ) {
            #Oops couldn't find  a server
            report "Couldn't find an available server.";
            return $_false;
    }

        # If we found a server, add the required number of sessions to
        # its allocation
        $servertable->[$lowest][1] += $count;
    $server = $servertable->[$lowest][0];
    verbose "Setting $server to $servertable->[$lowest][1] sessions.";
        
        # If we are trying to deallocate. If we have more than one session
        } else {
        # we will deallocate where we can
        my $found = $_false;
        for ( my $donecount = 0; $donecount < $count; $donecount ++) {

            $found = $_false;
            foreach $row ( @{$servertable} ) {

                # Loop if the server is disabled
                verbose "$row->[0] disabled." if ( $row->[2] eq "disabled" );
                next if ( $row->[2] eq "disabled" );
            
                if ( $row->[0] eq $server and $row->[1] > 0 ) {
                    $found = $_true;
                    $row->[1]--;
                    verbose "Setting $server to $row->[1] sessions.";
                    last;
                }
            }
        }
        if ( not $found ) {
            # We report that we couldn't complete de-allocation but don't fail
            report "Couldn't deallocate all sessions for $server.";
        }
        $server = ""
    }

    # write updated allocation table to file
        writeAllocFile ( $fh, $fname, $servertable );

    # close  server allocation file (automatically releases all lock)
    close $fh;
    unlink ( $locklog );
    verbose "Closed server allocaton file (lock released).";

    # set passed in server to allocated server / blank if de-allocated session
    $_[2] = $server;
    
    if ( $getserver ){ 
        
        report "allocated session to run on server $server (ok)";
    }
    else {
        
        report "de-allocated session from server $oldserver (ok)";
    }

    return $_true; 
}

# ------------------------------------------------------------------------------
# Allocate or de-allocate a session to a user in the user sessions file
sub userAssign($;$$) {
    my ($fname, $user, $userwanted) = @_;
    my $olduser = $user;
    my $row;
    
    # set getuser flag according to operation (allocate or de-allocate session)
    my $getuser = $_true;			# allocate
    $getuser = $_false if ( $user ne "" );	# de-allocate

    report  (	"trying to " .  ( ($getuser) ? "" : "de-" ) . 
		"allocate session for user $user" );

    # open user allocation file
    my $fh = new IO::File ( "$fname", "r+" )
	or die "Couldn't open file $fname ($!).";

    $fh->autoflush( 1 );

    verbose "Trying to get write-lock on user allocation file: $fname.";
    
    my $locklog = "$_locklog.user";

    # try to get exclusive lock on user allocation file
    unless ( lockfile ( $fh ) ) {
	report "Couldn't lock user allocation file: $fname.";
	showlock( $locklog );
	die "Couldn't lock user allocation file: $fname.";
    }

    loglock ( $locklog, $fname );

    verbose "Write-locked user allocation file (ok).";

    # read file into user table array
    my $usertable = readAllocFile ( $fh, $fname );
    # Maxsessions must be the first line in the table
    die "Couldn't find maxsessions in user allocation file ($fname)." 
	if ( $usertable->[0][0] ne "(maxsessions)" );
    my $maxsessions = $usertable->[0][1];

    # try to allocate / de-allocate session against specified user
    my $found = $_false;
    foreach $row ( @{$usertable} ) {

        # Loop if the row is the maxsessions line
	verbose "Maxsessions line" if ( $row->[0] eq "(maxsessions)" );
	next if ( $row->[0] eq "(maxsessions)" );
	# Loop if this user has been disabled
	verbose "$row->[0] disabled." if ( $row->[2] eq "disabled" );
	next if ( $row->[2] eq "disabled" );
	# Loop if a specific user is wanted and this isn't it
	verbose "$row->[0] is not reqd user $userwanted."
	    if ( $userwanted and $row->[0] ne $userwanted );
	next if ( $userwanted and $row->[0] ne $userwanted );
    	
	# If we're allocating a user...
	if ( $getuser ) {
	    if ( $row->[1] < $maxsessions ) {
		$found = $_true;
		$user = $row->[0];
		$row->[1]++;
		verbose "setting $row->[0] to $row->[1] sessions";
		last;
	    }
	} else { # If we're deallocating a user...
	    if ( $row->[0] eq $user and $row->[1] > 0 ) {
		$found = $_true;
		$user = "";
		$row->[1]--;
		verbose "setting $row->[0] to $row->[1] sessions";
		last;
	    }
	}
    }
    
    if ( not $found ) {
	# If we couldn't get an allocation then fail - without writing
	# the allocation table
	if ( $getuser ){ 
	    report "Couldn't find a free user session.";
	    return $_false;
	} else {
	# If we couldn't deallocate the used session, raise a warning
	# and write back what we have deallocated
	    report "Couldn't deallocate session for $user. Continuing...";
	}
    }

    # write updated allocation table to file
    writeAllocFile ( $fh, $fname, $usertable );

    # close  user allocation file (automatically releases all lock)
    close $fh;
    unlink ( $locklog );

    verbose "Closed user allocaton file (lock released)";

    # set passed in username to allocated user / blank if de-allocated session
    $_[1] = $user;
    
    if ( $getuser ){ report "Allocated session to user $user (ok)" }
    else { report "De-allocated session from user $olduser (ok)" }

    return $_true; 
}

# ------------------------------------------------------------------------------
# Read allocation file into an array (used for user and server allocation files)
sub readAllocFile($$) {
    my ($fh, $fname) = @_;
    my @table;
    my $maxsessions;

    verbose "reading allocation file $fname";

    my $line = 0;
    while ( <$fh> ) {
	chomp;
	$line++;
	verbose "\tline $line: $_";

	#my ($name, $sessions, $enabled) = split /:/;
	push @table, [ split /:/ ] ;
    }

    verbose "read allocation file into internal array";

    return ($maxsessions, \@table);
}

# ------------------------------------------------------------------------------
# Write allocation array to file (used for server and user allocation files)
sub writeAllocFile($$$) {
    my ($fh, $fname, $table) = @_;
    my $row;
    my $txtout;
    
    verbose "writing allocation file $fname";

    # go to beginning of file
    seek ( $fh, 0, 0 );

    my $line = 0;

    foreach $row ( @{$table} ) { 

	$line++;
	$txtout = join(":",@{$row}); 
	print $fh "$txtout\n"; 
	verbose "\tline $line: $txtout";
    }

    # chop off any remaining lines
    truncate ( $fh, tell ( $fh ) );

    verbose "written internal array to allocation file";

    return 1;
}

# ------------------------------------------------------------------------------
# Raise SNMP trap
sub snmpTrap($$) {
    my ($trapid, $text) = @_;
    my ($envfile, $bindir, $snmpexe);

    # don't try to riase another snmp if fatal error occurs during this routine
    my $old_send_traps = $_send_traps;
    $_send_traps = $_false;
    my($sendtrap);

    $_tab++;

    # get the open view snmp directories
    if ( defined $_profile ) {

	$envfile = getcfg ( "ERROR_HANDLING/OVENV"    );
	$bindir  = getcfg ( "ERROR_HANDLING/OVBIN"    );
	$snmpexe = getcfg ( "ERROR_HANDLING/SNMPTRAP" );
	$sendtrap = getcfg ( "ERROR_HANDLING/SENDTRAP" );
    }

    report "trap id is empty, using hardcoded default" if ( $trapid eq "" );
    
    # check for blank values from config, this would mean config profile not
    # loaded, so set to hardcoded defaults to allow snmp to be raised
    $trapid  = ( $trapid  eq "" ) ? $_def_trapid  : $trapid;
    $envfile = ( $envfile eq "" ) ? $_def_envfile : $envfile;
    $bindir  = ( $bindir  eq "" ) ? $_def_bindir  : $bindir;
    $snmpexe = ( $snmpexe eq "" ) ? $_def_snmpexe : $snmpexe;
    $sendtrap = ( $sendtrap eq "" ) ? $_def_sendtrap : $sendtrap;
    
    $envfile =~ s/%hostname%/$_hostname/;

    #verbose "snmp environment file: $envfile";
    #verbose "snmp executable: $bindir/$snmpexe";

    #my $envfh = new IO::File ( $envfile, "r" )
	#or die "couldn't open the snmp environment set-up file ($envfile)";

    #my %snmpenv;

    # read in snmp environment variables
    #while ( <$envfh> ) {
	#chomp;
	#if ( /^(.+)=(.+)$/ ) {
	#    my ($var, $val) = split /=/; 
	#    $snmpenv{$var} = $val; 
	#    verbose "snmp environment var $var=$snmpenv{$var}";
	#}
    #}
    
    # set-up snmp arguments
    #my @args;
    #push @args, $snmpenv{'OVNODE'};
    #push @args, $snmpenv{'OVENTERPRISE'};
    #push @args, $_hostname;
    #push @args, $trapid;
    #push @args, "\"\"";
    #push @args, $snmpenv{'OVENTERPRISE'};
    #push @args, "octetstringascii";
    #push @args, $text;
    
    # call snmptrap with correct params 
    #( execute ( "$bindir/$snmpexe" , \@args ) == $_success ) 
    #    or die "couldn't raise SNMP trap";

    my($cmd) = "$sendtrap \"$trapid\" $text 2>&1; echo REPLY=\$?";
    report "$cmd";
    my($line);
    my($okay) = $_false;
    open EXEC, "$cmd |";
    while ($line = <EXEC>) {
        $line =~ s/\s+$//;
        if ($line =~ m/REPLY=0/) {
           $okay = $_true;
        } else {
           report $line; 
        }
    }
    close EXEC;
    if (! $okay) {
       die "couldn't raise SNMP trap";
    }

    report "sent SNMP trap (id $trapid): $text";

    $_tab--;

    $_send_traps = $old_send_traps;

    return 1;
}

# ------------------------------------------------------------------------------
sub send_email($) {
        my($text) = @_;

        my $mailing_list      = getcfg("ERROR_HANDLING/POSTPROC_HANDLING/MAILING_LIST");
        my $mailing_from      = getcfg("ERROR_HANDLING/POSTPROC_HANDLING/MAILING_FROM");
        my $production_domain = getcfg("ERROR_HANDLING/PRODUCTION_DOMAIN");
        my $jobcode           = $_optctl{'jobcode'};
    my $subject           = "$jobcode FAILED";

    $text =~ s/^"//;
    $text =~ s/"$//;
        report "sending mail: $text";

        # test if email is coming from production or test environment
        my $domainname   = `/usr/bin/domainname`;
    $domainname =~ s/\s*$//;
    if ( $domainname ne $production_domain ) {
       report "Production domain is $production_domain, this domain is $domainname";
       my($hostname)    = `/usr/bin/uname -n`;
       $hostname =~ s/\s*$//;
       $subject = "$hostname:$subject";
       $text = "Message sent from server $hostname.$domainname \n\n" . $text;
    }

    if ( $mailing_from ne "" ) {
       $mailing_from = "-r $mailing_from";
    }

        # send email
        `/bin/mailx -s "$subject" $mailing_from $mailing_list <<EOD
$text
EOD`;
}

# ------------------------------------------------------------------------------
# Determine if current time is within the WORKING_HOURS
sub working_hours() {
   my($currmin, $currhour, $currday) = (localtime())[1,2,6];
   my($weekday) = ('SUNDAY','MONDAY','TUESDAY','WEDNESDAY','THURSDAY','FRIDAY','SATURDAY')[$currday];
   my($working_day) = getcfg ("ERROR_HANDLING/POSTPROC_HANDLING/WORKING_HOURS/$weekday");

   if ( length($working_day) == 0 ) { return $_false; } # non-working day

   my($starthour,$startmin, $endhour, $endmin);
   if (!(($starthour,$startmin, $endhour, $endmin) = $working_day =~ m/(\d\d):(\d\d)[\s-]+(\d\d):(\d\d)/)) {
      report "$weekday for WORKING_HOURS is incorrectly formatted in configuration file\n"; 
      return $_false; # faulty declaration
   }

   my($currallmins)  = $currhour * 60 + $currmin;
   my($startallmins) = $starthour * 60 + $startmin;
   my($endallmins)   = $endhour * 60 + $endmin;
   if (($currallmins >= $startallmins) && ($currallmins <= $endallmins)) {
      return $_true;
   } else {
      return $_false;
   }
}

# ------------------------------------------------------------------------------
# Get the error handling config
sub getErrConfig($) {
    my $level	= shift;
    my $bkey	= "ERROR_HANDLING/POSTPROC_HANDLING";
    
    # determine the severity level
    my $severity = "DEF";
    my $sevlist	 = getKeys ( "$bkey/SEVERITY" );
    
    foreach ( @$sevlist ) {
	
	my $severitydef = getcfg ( "$bkey/SEVERITY/$_" );
	my ($min, $max) = split /,/, $severitydef;
	
	if ( limitcheck ( $level, $min, $max ) ) { $severity = $_; last }
    }

    # determine the timezone
    my $timezone = "DEF";
    if ((working_hours()) && (getcfg("$bkey/ACTIONS/WORKING_TIME/DEF/SNMP"))) {
       $timezone = "WORKING_TIME";
    } else {

       my $timelist = getKeys ( "$bkey/TIMEZONES" );
    
       foreach ( @$timelist ) {
	
          my $timezonedef = getcfg ( "$bkey/TIMEZONES/$_" );
	      my ($hr1, $min1, $hr2, $min2) = split /[:-]/, $timezonedef;
	
	      my $timemin = $hr1*60 + $min1;
	      my $timemax = $hr2*60 + $min2;
	      my $timenow = timestamp("%H")*60 + timestamp("%M");

	      if ( between ( $timenow, $timemin, $timemax ) ) { $timezone = $_; last }
       }
    }

    # determine the actions to take
    my $snmp = getcfg ( "$bkey/ACTIONS/$timezone/$severity/SNMP" );
    my $stop = getcfg ( "$bkey/ACTIONS/$timezone/$severity/STOP" );

    unless ( $stop eq "YES" or $stop eq "NO" ) {
	
	$stop = "YES";
	
	report "STOP flag not recognised ($stop) for config key " .
	       "($bkey/ACTIONS/$timezone/$severity/STOP), assuming stop is YES";
    }

    if ( $snmp eq "" ) {
	
	report "SNMP key is empty for config key ($bkey/ACTIONS/$timezone" .
	       "/$severity/STOP), hardcoded default will be used instead";
    }

    my $email = getcfg ( "$bkey/ACTIONS/$timezone/$severity/EMAIL" );
    if ( $email eq "" ) {
       $email = getcfg ( "$bkey/ACTIONS/$timezone/DEF/EMAIL" );
    }
    if ($email ne "YES" and $email ne "NO") { $email = "NO"; }

    verbose "timezone ($timezone), severity ($severity), " . 
            "snmp key ($snmp), stop flag ($stop) email ($email)";


    return ($snmp, $stop, $email);
}

# ------------------------------------------------------------------------------
# Return the requested keys beneath the specified field in the config file
sub getKeys($) {
    my $field = shift;

    $field = "/SCB/POS_EOD/". $field unless $field =~ /^\/(.*)/;
    
    my @keys = $_profile->get_keys ( $field );

    return \@keys;
}

# ------------------------------------------------------------------------------
# Return the requested value from the config file
sub getcfg($;$) {
    my ($field, $nosubst) = @_;
    my $depth = 0;

    $field = "/SCB/POS_EOD/". $field unless $field =~ /^\/(.*)/;

    # extract the requested value
    my $value = $_profile->get_value ( $field );

    # assign an empty value if none was defined
    $value = "" unless defined $value;

    $value = clean ( $value );
    
    unless ( defined $nosubst ) {

	# environment variable substitution
	while ( $value =~ /\$(\w+)/ ) {

	    $value =~ s/\$(\w+)/$ENV{$1}/g;

	    die "cannot resolve nested environment variable in key $field"
		unless ( $depth++ < 100 );
	}
    }

    return $value;
}

# ------------------------------------------------------------------------------
# Return the xml template line with correct processing script values substituted
sub substPsCfg($$) {
    my ($line, $basekey) = @_;

    # config placeholder substitution
    while ( $line =~ /\%(\w+)\%/ ) {

	my $cfgitem = uc($1);
	my $value = getcfg ( "$basekey/$cfgitem" );
	$line =~ s/\%$1\%/$value/g;
    }
    
    return $line;
}

# ------------------------------------------------------------------------------
# Gets the business processing date
sub busdate($;$) {
    my ($delim, $type) = @_;
    my ($year, $month, $day);
    
    $type = "previous" unless ( defined $type );

    die "can't extract date for type ($type)" 
	unless ( $type eq "current" or $type eq "previous"
	or $type eq "last" or $type eq "last-1" );
	
    my $datefile = getcfg ( "MXG_DIR" ) . "/scb/reports/dates";

    if ( $type eq "last" or $type eq "last-1" ) {
	$datefile .= "/date_la.txt";
    }
    else {
	$datefile .= "/date_td.txt";
    }
	
    my $ifh = new IO::File ("$datefile", "r")
	or die "unable to open date file $datefile: $!";
    
    my $line = <$ifh>; $ifh->close();

    if ( $type eq "previous" or $type eq "last-1") {
        $year   = substr ( $line, 9, 4 );
	$month  = substr ( $line, 14, 2 );
	$day    = substr ( $line, 17, 2 );
    }
    else {
        $year   = substr ( $line, 28, 4 );
	$month  = substr ( $line, 33, 2 );
	$day    = substr ( $line, 36, 2 );
    }

    return "$year$delim$month$delim$day"
}

# ------------------------------------------------------------------------------
# Log details of file just locked 
sub loglock($$) {
    my ($locklog, $fname) = @_;

    my $fh = new IO::File ( $locklog, "w" )
	or die "couldn't open the flock log: $locklog";
    
    print $fh "file:"		. $fname;
    print $fh "\nlock_time:"	. localtime();
    print $fh "\npid:"		. $PID;
    print $fh "\ncmd:"		. $PROGRAM_NAME;
    print $fh "\nargs:"; 
    
    foreach ( keys %_optctl ) { print $fh "-$_ $_optctl{$_} " }
    
    print $fh "\nuid_eff:"	.   $EFFECTIVE_USER_ID;
    print $fh "\nuid_real:"	.   $REAL_USER_ID;

    close $fh;
}

# ------------------------------------------------------------------------------
#  Print details of lock log to stdout
sub showlock($) {
    my $locklog = shift;
    
    my $fh = new IO::File ( $locklog, "r" )
	or die "couldn't open the flock log: $locklog";
    
    report "contents of lock log: $locklog";

    while ( <$fh> ) { 
	chomp;
	report "\t$_"
    }

    report "end of contents of lock log: $locklog";
    close $fh;
}

# ------------------------------------------------------------------------------
# Get the current recorded duration
sub getduration() {
    my $duration = $_timer->duration("nop");

    my $secs = $1 if ($duration =~ /\s*(\d+) wallclock/);

    return $secs;
}

1;

#-- EOF


## balance
+ /shared/opt/SCB/pos_eod/live/scripts/eod_task.pl -jobcode DSVAR03 -config irdfxdev9 
BASEDIR=/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0
2016_06_18 07:09:37 Loading job definition jobdefs/dsvar03.cf
2016_06_18 07:09:37 running job DSVAR03 (Sat 18 Jun 2016 @ 0709_37) on magapp15a (pid=21574)
2016_06_18 07:09:37 business processing date is 2015_04_30
2016_06_18 07:09:37 SNMP traps are ON
2016_06_18 07:09:37 post-processing checks are ON
2016_06_18 07:09:37 running job commands...
2016_06_18 07:09:37     trying to allocate session for user 
2016_06_18 07:09:37     Allocated session to user VARCON1 (ok)
2016_06_18 07:09:37     trying to allocate server 
2016_06_18 07:09:37     allocated session to run on server MAGAPP15A (ok)
2016_06_18 07:09:37     running cmd:
2016_06_18 07:09:37         cd /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client ; /shared/opt/jdk/pos/live/jre/bin/java
2016_06_18 07:09:37         -cp mxjboot.jar
2016_06_18 07:09:37         -Djava.security.policy=java.policy
2016_06_18 07:09:37         -Djava.rmi.server.codebase=http://uksvadmrx02a.uk.standardchartered.com:60001/murex.download.guiclient.\
download murex.rmi.loader.RmiLoader
2016_06_18 07:09:37         /MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript
2016_06_18 07:09:37         /MXJ_SITE_NAME:mxg_irdfx_dev9
2016_06_18 07:09:37         /MXJ_PLATFORM_NAME:MX
2016_06_18 07:09:37         /MXJ_PROCESS_NICK_NAME:POS_EOD_MXPS_MAGAPP15A_VARCON1
2016_06_18 07:09:37         /MXJ_CONFIG_FILE:/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_37.DSV\
AR03.psr/xml_request.xml
2016_06_18 07:09:37     end of cmd
2016_06_18 07:09:37     cmd std out:
2016_06_18 07:09:38         Opening bin/file.version...
2016_06_18 07:09:38         Opening jar/file.version...
2016_06_18 07:09:38         Checking files...
2016_06_18 07:09:38         bin/file.version saved.
2016_06_18 07:09:38         jar/file.version saved.
2016_06_18 07:09:39         Loading public/mxres/common/properties.mxres from http://uksvadmrx02a.uk.standardchartered.com:60001/mu\
rex.download.guiclient.download
2016_06_18 07:09:39         Loading locally /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_37.DSVA\
R03.psr/xml_request.xml.
2016_06_18 07:09:39         Loading public/mxres/sites/sites.mxres from http://uksvadmrx02a.uk.standardchartered.com:60001/murex.do\
wnload.guiclient.download
2016_06_18 07:09:39         murex.xml.server.home.balancing.exception.BalancingHomeException: Error while creating process for sess\
ion.Unable to find launcher for [MX,,POS_EOD_MXPS_MAGAPP15A_VARCON1]
2016_06_18 07:09:39             at murex.xml.server.home.balancing.BalancingLogic.logNotFoundLauncher(BalancingLogic.java:402)
2016_06_18 07:09:39             at murex.xml.server.home.balancing.BalancingLogic.createProcessForSession(BalancingLogic.java:109)
2016_06_18 07:09:39             at murex.xml.server.home.balancing.BalancingHome.createProcessForSession(BalancingHome.java:136)
2016_06_18 07:09:39             at murex.xml.server.home.XmlHome.createSessionLocally(XmlHome.java:592)
2016_06_18 07:09:39             at murex.xml.server.home.XmlHome.createSession(XmlHome.java:559)
2016_06_18 07:09:39             at sun.reflect.GeneratedMethodAccessor7.invoke(Unknown Source)
2016_06_18 07:09:39             at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
2016_06_18 07:09:39             at java.lang.reflect.Method.invoke(Method.java:324)
2016_06_18 07:09:39             at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:261)
2016_06_18 07:09:39             at sun.rmi.transport.Transport$1.run(Transport.java:148)
2016_06_18 07:09:39             at java.security.AccessController.doPrivileged(Native Method)
2016_06_18 07:09:39             at sun.rmi.transport.Transport.serviceCall(Transport.java:144)
2016_06_18 07:09:39             at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:460)
2016_06_18 07:09:39             at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:701)
2016_06_18 07:09:39             at java.lang.Thread.run(Thread.java:534)
2016_06_18 07:09:39     end of cmd std out
2016_06_18 07:09:39     cmd cd /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client ; /shared/opt/jdk/pos/live/jre/bin/java: ran with no\
rmal exit
2016_06_18 07:09:39     trying to de-allocate session for user VARCON1
2016_06_18 07:09:39     De-allocated session from user VARCON1 (ok)
2016_06_18 07:09:39     trying to de-allocate server MAGAPP15A
2016_06_18 07:09:39     de-allocated session from server MAGAPP15A (ok)
2016_06_18 07:09:39 end of job commands (last return code = 0)
2016_06_18 07:09:39 running post-processing checks for DSVAR03
2016_06_18 07:09:39     running level 1 check: AC1 fileExists ( '/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30\
/eod_job.0709_37.DSVAR03.psr/answer.xml'  )
2016_06_18 07:09:39     end of check (ok)
2016_06_18 07:09:39     running level 2 check: AC2 searchFile ( '/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30\
/eod_job.0709_37.DSVAR03.psr/answer.xml' 'present' '-1' '-1' 'Ended_Successfully'  )
2016_06_18 07:09:39         ERR: Batch failed. See /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_\
37.DSVAR03.psr/answer.xml.
2016_06_18 07:09:39         no SNMP rasied
2016_06_18 07:09:39     end of check (failed)
2016_06_18 07:09:39     no SNMP rasied
2016_06_18 07:09:39     sending mail: job DSVAR03 (started on Sat 18 Jun 2016 @ 0709_37) failed level 2 check [ AC2 searchFile ( '/\
shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_37.DSVAR03.psr/answer.xml' 'present' '-1' '-1' 'Ende\
d_Successfully'  ) ] see logfile [ eod_job.0709_37.DSVAR03.log ] err [ Batch failed. See /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/c\
lient/runtime/2015_04_30/eod_job.0709_37.DSVAR03.psr/answer.xml. ] feedback [ ]
2016_06_18 07:09:39     Production domain is gdc.standardchartered.com, this domain is uk.standardchartered.com
2016_06_18 07:09:39 end of post-processing checks for DSVAR03 (failed)
2016_06_18 07:09:39 --- 1 -----------------------
"job DSVAR03 (started on Sat 18 Jun 2016 @ 0709_37) failed level 2 check [ AC2 searchFile ( '/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1\
.0/client/runtime/2015_04_30/eod_job.0709_37.DSVAR03.psr/answer.xml' 'present' '-1' '-1' 'Ended_Successfully'  ) ] see logfile [ eo\
d_job.0709_37.DSVAR03.log ] err [ Batch failed. See /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709\
_37.DSVAR03.psr/answer.xml. ] feedback [ ]"
Error Counts -----------------------------------
Critical: 1
All:      1

2016_06_18 07:09:39 end of job DSVAR03 (failed)
2016_06_18 07:09:39 job DSVAR03 took  2 wallclock secs ( 0.02 usr  0.05 sys +  1.00 cusr  0.31 csys =  1.38 CPU)


## ## putty ##
Mai_title: Emailing: putty download method.zip  / FW: Tools Share 

## Aqua Data Studio ##



## Set up VPN software client## 
Mail_Title: Follow Up - 8308129-001-201 - VPN Setup

## Set up Printer Driver## 
Mail_Title: RMS# 8327306-007-101 add printer driver
Doc: Job\Document\Set_up_printer

## Outlook ## Add WenJun & Tony ( Zhu Bing ) into mail grp 
Firstly make sure you have store the group as your contact, then open the group contact and then double click the group name and pop out a pannel and then click the 'modify number'.

 
## Timesheet ## Submit timesheet in WorkWise.
Doc: Job\Document\TimeSheet
	

## Self-Regist in JIRA ##
https://jira.uk.standardchartered.com:8443/secure/Dashboard.jspa (Note visit it with Chrome not IE,and the url is incorrect in Job\Document\SetupDevelopmentEnvironment\New Joiners.docx)  


## VPN don't work ##
check Preferences->Block connections to untrusted server(check out this item)


## Control-M ????????????????????????????
ITIL(Information Technology Infrastructure Library),信息技术基础架构库，由英国政府部门CCTA(Central Computing and Telecomunications Agency)在20世纪80年代末制定，现由英国商务部（0
BSM(Business Service Management),业务服务管理，
Control-M production is developed by BMC software Corporation ,


## Excel - vlookup ??????????????????????



## murex client 
1)bin  文件夹
  avsjogl.dll
  xxx.dll
  bridge2mx.tlb
  excel2mx.tlb
  file.version
  xxx.dll
  regtlb.exe
2)jar 文件夹
 　sun.jar
  com.ibm.mq.jar
  com.ibm.mqbind.jar
  com.ibm.mqjms.jar
  excel.jar
  ie.jar
  file.version
  jta.jar
  mail.jar
  mxj.jar
  servlet.jar
  word.jar
  xml4.jar
  xxxx.jar

3)mxjboot.jar
4)client_mxg_qa1.bat
  
@ECHO OFF

REM Mx G2000 Client Launcher
REM Mofify this script to match your java and server environnement
REM For 2.2.8 and 2.2.9
REM V2.3

setlocal

SET JAVAHOME=%ProgramFiles(x86)%\Java\jre1.5.0_15

SET MXJ_FILESERVER_HOST=magapp11a.uk.standardchartered.com
SET MXJ_FILESERVER_PORT=14331
SET MXJ_SITE_NAME=default
SET MXJ_DESTINATION_SITE_NAME=mxg_qa1
SET MXJ_PLATFORM_NAME=MX
SET MXJ_PROCESS_NICK_NAME=MXG_QA1


SET PATH=%JAVAHOME%\jre\bin;%JAVAHOME%\jre\bin\classic;%JAVAHOME%\bin;%JAVAHOME%\bin\classic;%PATH%

SET PATH=%PATH%;bin
SET MXJ_JAR_FILELIST=murex.download.guiclient.download
SET MXJ_POLICY=java.policy
SET MXJ_BOOT=mxjboot.jar
SET MXJ_CONFIG_FILE=client.xml

IF EXIST jar\%MXJ_BOOT% copy jar\%MXJ_BOOT% . >NUL

title %~n0 FS:%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST%  Xml:%MXJ_SITE_NAME% /PLATF:%MXJ_PLATFORM_NAME% /NNAME:%MXJ_PROCESS_NICK_NAME%

java -Xmx512m -cp %MXJ_BOOT% -Djava.security.policy=%MXJ_POLICY% -Djava.rmi.server.codebase=http://%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST% murex.rmi.loader.RmiLoader /MXJ_SITE_NAME:%MXJ_SITE_NAME% /MXJ_DESTINATION_SITE_NAME:%MXJ_DESTINATION_SITE_NAME% /MXJ_CLASS_NAME:murex.gui.xml.XmlGuiClientBoot /MXJ_PLATFORM_NAME:%MXJ_PLATFORM_NAME% /MXJ_PROCESS_NICK_NAME:%MXJ_PROCESS_NICK_NAME% /MXJ_CONFIG_FILE:%MXJ_CONFIG_FILE% %1 %2 %3 %4 %5 %6

title Command Prompt
endlocal
pause

## mxjboot.jar
package murex.rmi.loader.parser.sax;

class Attribute
{
  private String name;
  private String value;

  public Attribute(String n, String v)
  {
    this.name = n;
    this.value = v;
  }

  public String getName()
  {
    return this.name;
  }

  public String getValue()
  {
    return this.value;
  }
}

package murex.rmi.loader.parser.sax;

import java.util.Vector;

public class AttributeList
{
  private Vector attributes;

  public AttributeList()
  {
    this.attributes = new Vector();
  }

  public void addAttribute(String name, String value)
  {
    Attribute attr = new Attribute(name, value);
    this.attributes.addElement(attr);
  }

  public int getLength()
  {
    return this.attributes.size();
  }

  public String getName(int index)
  {
    return ((Attribute)this.attributes.elementAt(index)).getName();
  }

  public String getValue(String n)
  {
    int i = getIndex(n);
    if (i != -1) {
      return getValue(i);
    }
    return null;
  }

  public String getValue(int index)
  {
    return ((Attribute)this.attributes.elementAt(index)).getValue();
  }

  public String getType(int index)
  {
    return null;
  }

  public String getType(String n)
  {
    return null;
  }

  public void clear()
  {
    this.attributes.removeAllElements();
  }

  private int getIndex(String n)
  {
    int index = -1;
    for (int i = 0; i < this.attributes.size(); i++)
      if (((Attribute)this.attributes.elementAt(i)).getName().equals(n))
        index = i;
    return index;
  }
}

package murex.rmi.loader.parser.sax;

public abstract interface HandlerBase
{
  public abstract void characters(char[] paramArrayOfChar, int paramInt1, int paramInt2);

  public abstract void startDocument();

  public abstract void endDocument();

  public abstract void startElement(String paramString, AttributeList paramAttributeList);

  public abstract void endElement(String paramString);
}

package murex.rmi.loader.parser.sax;

import java.io.InputStream;
import java.io.Reader;

public class InputSource
{
  private InputStream byteStream;
  private Reader characterStream;

  public InputSource(InputStream i)
  {
    this.byteStream = i;
  }

  public InputSource(Reader r)
  {
    this.characterStream = r;
  }
}

package murex.rmi.loader.parser.sax;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;

public class Parser
{
  private HandlerBase handler = null;
  private char[] input;
  private int cursor = 0;
  private int length = 0;
  private String elemName;
  private boolean endElem = false;
  private AttributeList attributes;

  public void setDocumentHandler(HandlerBase h)
  {
    this.handler = h;
  }

  public void parse(String str)
  {
    parse(str.toCharArray());
  }

  public void parse(Reader r) throws IOException
  {
    BufferedReader bf = new BufferedReader(r);

    String str = new String();
    String str1;
    while ((str1 = bf.readLine()) != null)
    {
      str1 = str1.trim();
      str = str + str1;
    }
    char[] ch = str.toCharArray();
    parse(ch);
  }

  public void parse(char[] data) {
    this.input = data;
    int prevCursor = 0;
    this.length = this.input.length;
    boolean foundChar = false;
    int charCur = 0;

    this.handler.startDocument();

    for (this.cursor = 0; this.cursor < this.length; this.cursor += 1) {
      switch (this.input[this.cursor]) {
      case '&':
        gotAmpersand();
        break;
      case '<':
        if (this.cursor > prevCursor)
          this.handler.characters(this.input, prevCursor, this.cursor - prevCursor);
        if ((this.input[(this.cursor + 1)] == '?') || (this.input[(this.cursor + 1)] == '!')) {
          ignore();
          prevCursor = this.cursor + 1;
        }
        else {
          gotLT();
        }break;
      case '>':
        foundChar = false;
        prevCursor = this.cursor + 1;
        gotGT();
        if (this.input[(this.cursor - 1)] == '/')
          this.handler.endElement(this.elemName); break;
      case '\n':
        prevCursor = this.cursor + 1;
      }

    }

    this.handler.endDocument();
  }

  private void gotLT()
  {
    this.attributes = null;
    boolean b = false;
    boolean gotName = false;
    int prevCursor = this.cursor + 1;

    this.endElem = false;
    for (this.cursor += 1; this.cursor < this.length; this.cursor += 1)
    {
      switch (this.input[this.cursor])
      {
      case '&':
        gotAmpersand();
        break;
      case '/':
        if (this.input[(this.cursor + 1)] != '>')
        {
          prevCursor++;
          this.endElem = true; } break;
      case '>':
        b = true;
        if (!gotName)
        {
          if (this.input[(this.cursor - 1)] != '/')
            this.elemName = new String(this.input, prevCursor, this.cursor - prevCursor);
          else
            this.elemName = new String(this.input, prevCursor, this.cursor - prevCursor - 1);
          gotName = true;
        }
        this.cursor -= 1;
        break;
      case ' ':
        if (!gotName)
        {
          this.elemName = new String(this.input, prevCursor, this.cursor - prevCursor);
          gotName = true;
        }
        gotAttribute();
      }

      if (b)
        break;
    }
  }

  private void gotAttribute() {
    if (this.attributes == null) {
      this.attributes = new AttributeList();
    }
    int prevCursor = this.cursor + 1;
    int i = 0;
    boolean b = false;

    for (this.cursor += 1; this.cursor < this.length; this.cursor += 1)
    {
      switch (this.input[this.cursor])
      {
      case '&':
        gotAmpersand();
        break;
      case '=':
        i = this.cursor;
        break;
      case ' ':
        b = true;
        this.cursor -= 1;
        break;
      case '>':
        b = true;
        this.cursor -= 1;
        if (this.input[this.cursor] == '/')
          this.cursor -= 1;
        break;
      }
      if (b)
        break;
    }
    String n = new String(this.input, prevCursor, i - prevCursor);
    String v;
    String v;
    if (this.cursor - i > 2)
      v = new String(this.input, i + 2, this.cursor - i - 2);
    else
      v = new String("");
    this.attributes.addAttribute(n, v);
  }

  private void gotGT()
  {
    if (!this.endElem)
      this.handler.startElement(this.elemName, this.attributes);
    else
      this.handler.endElement(this.elemName);
  }

  private void ignore()
  {
    for (this.cursor += 1; (this.cursor < this.length) && 
      (this.input[this.cursor] != '>'); this.cursor += 1);
  }

  private void gotAmpersand()
  {
    int i = 0;
    char[] ch = new char[5];
    while (this.input[(i + this.cursor + 1)] != ';')
    {
      ch[i] = this.input[(i + this.cursor + 1)];
      i++;
    }
    String str = new String(ch, 0, i);

    if (str.equals("amp"))
      this.input[this.cursor] = '&';
    if (str.equals("quot"))
      this.input[this.cursor] = '"';
    if (str.equals("lt"))
      this.input[this.cursor] = '<';
    if (str.equals("gt"))
      this.input[this.cursor] = '>';
    if (str.equals("apos"))
      this.input[this.cursor] = '\'';
    System.arraycopy(this.input, i + this.cursor + 2, this.input, this.cursor + 1, this.length - i - this.cursor - 2);
    this.length = (this.length - i - 1);
  }
}

package murex.rmi.loader.parser;

public class CheckFile
{
  private String strFile;
  private String strCheck;
  private String strVersion;
  private String strCheckSum;

  public CheckFile(String strFile, String strCheck, String strVersion, String strCheckSum)
  {
    this.strFile = strFile;
    this.strCheck = strCheck;
    this.strVersion = strVersion;
    this.strCheckSum = strCheckSum;
  }

  public String getCheck() {
    return this.strCheck;
  }

  public String getFile() {
    return this.strFile;
  }

  public String getVersion() {
    return this.strVersion;
  }

  public String getCheckSum() {
    return this.strCheckSum;
  }

  public String toString() {
    return "[File : " + this.strFile + (this.strCheck == null ? "" : new StringBuffer().append(" , Check : ").append(this.strCheck).toString()) + (this.strVersion == null ? "" : new StringBuffer().append(" , Version : ").append(this.strVersion).toString()) + (this.strCheckSum == null ? "" : new StringBuffer().append(" , CheckSum : ").append(this.strCheckSum).toString()) + "]";
  }
}

package murex.rmi.loader.parser;

public class FileVersion
{
  private String strFile;
  private String strVersion;
  private String strCheckSum;

  public FileVersion(String strFile, String strVersion, String strCheckSum)
  {
    this.strFile = strFile;
    this.strVersion = strVersion;
    this.strCheckSum = strCheckSum;
  }

  public String getFile() {
    return this.strFile;
  }

  public String getVersion() {
    return this.strVersion;
  }

  public String getCheckSum() {
    return this.strCheckSum;
  }

  public String toString() {
    return "[File : " + this.strFile + (this.strVersion == null ? " , CheckSum : " + this.strCheckSum : new StringBuffer().append(" , Version : ").append(this.strVersion).toString()) + "]";
  }
}

package murex.rmi.loader.parser;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.util.ArrayList;
import murex.rmi.loader.parser.sax.AttributeList;
import murex.rmi.loader.parser.sax.HandlerBase;

public class Parser
  implements HandlerBase
{
  public static final String tagPhysicalDownLoad = "PhysicalDownLoad";
  public static final String tagMemoryDownLoad = "MemoryDownLoad";
  public static final String tagClassFiles = "ClassFiles";
  public static final String tagCheck = "Check";
  public static final String tagFile = "File";
  public static final String tagVersion = "Version";
  public static final String tagCheckSum = "CheckSum";
  private murex.rmi.loader.parser.sax.Parser parser;
  private ArrayList physical;
  private ArrayList memory;
  private ArrayList currentDld;
  private String currentTag;
  private String currentFile;
  private String currentCheck;
  private String currentVersion;
  private String currentCheckSum;

  public Parser(String strAllFile)
    throws Exception
  {
    this.parser = new murex.rmi.loader.parser.sax.Parser();
    this.parser.setDocumentHandler(this);
    this.parser.parse(strAllFile);
  }

  public void characters(char[] ch, int start, int length) {
    this.currentFile = new String(ch, start, length);
  }

  public void startDocument() {
  }

  public void endDocument() {
  }

  public void startElement(String name, AttributeList attributes) {
    if (name.equals("PhysicalDownLoad")) {
      this.physical = new ArrayList();
      this.currentTag = "PhysicalDownLoad";
      this.currentDld = this.physical;
    } else if (name.equals("ClassFiles")) {
      this.memory = new ArrayList();
      this.currentDld = this.memory;
      this.currentTag = "MemoryDownLoad";
    } else if ((name.equals("File")) && 
      (attributes != null)) {
      this.currentCheck = attributes.getValue("Check");
      this.currentVersion = attributes.getValue("Version");
      this.currentCheckSum = attributes.getValue("CheckSum");
    }
  }

  public void endElement(String name)
  {
    if (name.equals("File"))
      this.currentDld.add(new CheckFile(this.currentFile, this.currentCheck, this.currentVersion, this.currentCheckSum));
  }

  public CheckFile[] getPhysicalDownload()
  {
    if (this.physical != null) {
      return (CheckFile[])this.physical.toArray(new CheckFile[0]);
    }
    return new CheckFile[0];
  }

  public CheckFile[] getMemoryDownload() {
    if (this.memory != null) {
      return (CheckFile[])this.memory.toArray(new CheckFile[0]);
    }
    return new CheckFile[0];
  }

  public static void main(String[] argv) {
    try {
      StringBuffer sb = new StringBuffer();
      BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(argv[0])));
      String line = br.readLine();
      while (line != null) {
        sb.append(line);
        line = br.readLine();
      }
      Parser parser = new Parser(sb.toString());

      System.out.println("PhysicalDownload:");
      CheckFile[] physical = parser.getPhysicalDownload();
      if (physical != null) {
        for (int i = 0; i < physical.length; i++) {
          System.out.println(physical[i]);
        }
      }
      System.out.println("MemoryDownload:");
      CheckFile[] memory = parser.getMemoryDownload();
      if (memory != null)
        for (int i = 0; i < memory.length; i++)
          System.out.println(memory[i]);
    }
    catch (Exception e) {
      e.printStackTrace();
    }
  }
}

package murex.rmi.loader.parser;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
import murex.rmi.loader.parser.sax.AttributeList;
import murex.rmi.loader.parser.sax.HandlerBase;
import murex.rmi.loader.parser.sax.Parser;

public class VersionParser
  implements HandlerBase
{
  public String tagFiles = "Files";
  public String tagFile = "File";
  public String tagVersion = "Version";
  public String tagCheckSum = "CheckSum";
  private Parser parser;
  private Map map;
  private String currentFile;
  private String currentVersion;
  private String currentCheckSum;

  public VersionParser(String strFile)
    throws Exception
  {
    StringBuffer sb = new StringBuffer();
    BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(strFile)));
    String line = br.readLine();
    while (line != null) {
      sb.append(line);
      line = br.readLine();
    }

    this.map = new TreeMap();
    this.parser = new Parser();
    this.parser.setDocumentHandler(this);
    this.parser.parse(sb.toString());
  }

  public void characters(char[] ch, int start, int l) {
    this.currentFile = new String(ch, start, l);
  }

  public void startDocument() {
  }

  public void endDocument() {
  }

  public void startElement(String name, AttributeList attributes) {
    if ((name.equals(this.tagFile)) && 
      (attributes != null)) {
      this.currentVersion = attributes.getValue(this.tagVersion);
      this.currentCheckSum = attributes.getValue(this.tagCheckSum);
    }
  }

  public void endElement(String name)
  {
    if (name.equals(this.tagFile)) {
      FileVersion fileVersion = new FileVersion(this.currentFile, this.currentVersion, this.currentCheckSum);
      this.map.put(fileVersion.getFile(), fileVersion);
      this.currentVersion = null;
      this.currentCheckSum = null;
    }
  }

  public Map getFileVersion() {
    return this.map;
  }

  public static void main(String[] argv) {
    try {
      VersionParser versionParser = new VersionParser(argv[0]);
      System.out.println("File : ");
      Map map = versionParser.getFileVersion();
      Iterator iterator = map.keySet().iterator();
      while (iterator.hasNext()) {
        String strFile = (String)iterator.next();
        System.out.println(map.get(strFile));
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}

package murex.rmi.loader.start;

import java.io.PrintStream;

public class FileServerAutomaticDownload
  implements Runnable
{
  public static void main(String[] args)
  {
    System.out.println("MX files were successfully downloaded...");
  }
  public void run() {
    main(null);
  }
}

package murex.rmi.loader.webstart;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLConnection;

public class BuildClientDir
{
  public static final String tagFile = "File";
  public static final String tagException = "EXCEPTION";
  public static final String tagMurexAnswer = "Murex-Answer";
  public static final String FILE_SERVER = "/FILE_SERVER=";
  public static final String FILE_SERVER_PORT = "/FILE_SERVER_PORT=";
  public static final String MXDOC_SERVER = "/MXDOC_SERVER=";
  public static final String MXDOC_SERVER_PORT = "/MXDOC_SERVER_PORT=";
  public static final String MXDOC_HTTP_PROXY_HOST = "/MXDOC_HTTP_PROXY_HOST=";
  public static final String MXDOC_HTTP_PROXY_PORT = "/MXDOC_HTTP_PROXY_PORT=";
  public static final String MXDOC_PROXY_AUTHENTIFICATION_NAME = "/MXDOC_PROXY_AUTHENTIFICATION_NAME=";
  public static final String MXDOC_PROXY_AUTHENTIFICATION_PASSWD = "/MXDOC_PROXY_AUTHENTIFICATION_PASSWD=";
  public static final String CLIENT_DIR = "/CLIENT_DIR=";
  public static final String JAVA_SECURITY_POLICY = "-Djava.security.policy=";
  public static final String RMI_SERVER_CODE_BASE = "-Djava.rmi.server.codebase=";
  public static final String MXJ_BOOT_URL_FROM_FS = "webclient/jar/mxjboot.jar";
  public static final String MXJ_MXDOC_BOOT_URL_FROM_FS = "webclient/mxdoc/mxjboot.jar";
  public static final String MXJ_JAVA_HOME = "JAVA_HOME=";
  public static final String MXJ_JAVA_MAX_HEAP = "JAVA_MAX_HEAPSIZE=";
  public static final String MXJ_JAVA_MIN_HEAP = "JAVA_MIN_HEAPSIZE=";

  public static void main(String[] argv)
  {
    if (argv.length >= 16)
      try {
        BuildClientDir buildClientDir = new BuildClientDir();
        buildClientDir.getBoot(argv);
        buildClientDir.getMxDocBoot(argv);
        File clientCmdFile = buildClientDir.createClientCmdFile(argv);
        buildClientDir.createMxDocCmdFile(argv);
        buildClientDir.launchWebClient(clientCmdFile);
      } catch (Exception e) {
        e.printStackTrace();
      }
    else
      System.out.println("Please Enter the minimum number of arguments in the JNLP file...");
  }

  private void getBoot(String[] strArgs)
    throws Exception
  {
    String fileServerHost = getArgument(strArgs, "/FILE_SERVER=");
    String fileServerPort = getArgument(strArgs, "/FILE_SERVER_PORT=");
    String jnlpClientPath = getArgument(strArgs, "/CLIENT_DIR=");
    String clientPath = computeClientDirPath(jnlpClientPath);
    System.out.println("/FILE_SERVER=" + fileServerHost);
    System.out.println("/FILE_SERVER_PORT=" + fileServerPort);
    System.out.println("/CLIENT_DIR=" + clientPath);

    String strMxjBootUrl = "http://" + fileServerHost + ":" + fileServerPort + "/" + "webclient/jar/mxjboot.jar";
    URL servUrl = new URL(strMxjBootUrl);
    try
    {
      jarFile = getFileFromFileServer(servUrl);
    }
    catch (Exception e)
    {
      byte[] jarFile;
      e.printStackTrace(System.err);
      throw e;
    }
    byte[] jarFile;
    File clientPathFile = new File(clientPath);
    if (!clientPathFile.exists()) {
      if (clientPathFile.mkdirs())
        System.out.println("The folder CLIENT_DIR has been created: " + clientPathFile);
      else {
        throw new Exception("Failed to create CLIENT_DIR: " + clientPathFile);
      }
    }

    String destJarFile = clientPath + "\\mxjboot.jar";
    createMxjBoot(jarFile, destJarFile);
  }

  private String computeClientDirPath(String jnlpPath) throws Exception
  {
    String returnedString = null;
    String rootClientPath = null;
    String userName = System.getProperty("user.name");
    if (jnlpPath.indexOf("\\USER.NAME") == -1)
    {
      returnedString = jnlpPath;
    }
    else {
      rootClientPath = jnlpPath.substring(0, jnlpPath.indexOf("\\USER.NAME"));
      returnedString = rootClientPath + "\\" + userName;
    }
    return returnedString;
  }

  private void getMxDocBoot(String[] strArgs) throws Exception {
    String fileServerHost = getArgument(strArgs, "/FILE_SERVER=");
    String fileServerPort = getArgument(strArgs, "/FILE_SERVER_PORT=");
    String jnlpClientPath = getArgument(strArgs, "/CLIENT_DIR=");
    String clientPath = computeClientDirPath(jnlpClientPath);
    String mxDocClientPath = clientPath + "\\mxdoc";

    String strMxjMxDocBootUrl = "http://" + fileServerHost + ":" + fileServerPort + "/" + "webclient/mxdoc/mxjboot.jar";
    URL servUrl = new URL(strMxjMxDocBootUrl);
    try
    {
      jarFile = getFileFromFileServer(servUrl);
    }
    catch (Exception e)
    {
      byte[] jarFile;
      e.printStackTrace(System.err);
      throw e;
    }
    byte[] jarFile;
    File mxDocClientPathFile = new File(mxDocClientPath);
    if (!mxDocClientPathFile.exists()) {
      if (mxDocClientPathFile.mkdirs())
        System.out.println("The folder mxdoc has been created: " + mxDocClientPathFile);
      else {
        throw new Exception("Failed to create mxdoc: " + mxDocClientPathFile);
      }
    }

    String destJarFile = mxDocClientPath + "\\mxjboot.jar";
    createMxjBoot(jarFile, destJarFile);
  }

  private static byte[] getFileFromFileServer(URL url) throws Exception {
    String strRequest = url.getFile() + "?" + "File" + "=" + "webclient/jar/mxjboot.jar";
    URL urlToFileServer = new URL(url.getProtocol(), url.getHost(), url.getPort(), strRequest);
    URLConnection urlConnection = urlToFileServer.openConnection();
    urlConnection.setUseCaches(false);
    urlConnection.setRequestProperty("MUREX-EXTENSION", "murex/murex.rmi.loader.FileServerGenericLoader");
    InputStream inputStream = urlConnection.getInputStream();
    if (urlConnection.getContentLength() == -1) {
      throw new Exception("unable to get file webclient/jar/mxjboot.jar from " + url + ".");
    }

    String strStatus = urlConnection.getHeaderField("Murex-Answer");
    if ("EXCEPTION".equals(strStatus)) {
      byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
      throw new Exception(new String(inputFile));
    }
    byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
    inputStream.close();
    return inputFile;
  }

  private static byte[] getBodyStream(InputStream inputStream, int count) {
    byte[] tab = new byte[count];
    try {
      int iReceived = 0;
      int iTempReceived = 0;
      while ((iTempReceived = inputStream.read(tab, iReceived, count - iReceived)) > 0)
        iReceived += iTempReceived;
    }
    catch (IOException e) {
    }
    return tab;
  }

  private void createMxjBoot(byte[] byteFile, String strDestFile) {
    if (byteFile.length != 0)
      try {
        FileOutputStream fos = new FileOutputStream(strDestFile);
        fos.write(byteFile);
        fos.close();
        System.out.println(strDestFile + " copied.");
      } catch (Exception e) {
        System.out.println(e.getMessage());
      }
  }

  private static String getArgument(String[] strArgs, String neededArg)
  {
    for (int i = 0; i < strArgs.length; i++) {
      if (strArgs[i].startsWith(neededArg)) {
        return strArgs[i].substring(neededArg.length());
      }
    }
    return null;
  }

  public File createClientCmdFile(String[] args) throws Exception {
    String jnlpClientPath = getArgument(args, "/CLIENT_DIR=");
    String destDir = computeClientDirPath(jnlpClientPath);
    File webclientCmd = new File(destDir, "webclient.cmd");
    String javaHome = System.getProperty("java.home");
    String javaSecPolicy = "";
    String commandArgs = "";
    String rmiServerCodeBase = "";
    String strXmx = null;
    String strXms = null;

    for (int i = 0; i < args.length; i++) {
      String arg = args[i];
      if (arg.startsWith("-Djava.security.policy="))
        javaSecPolicy = arg;
      else if (arg.startsWith("-Djava.rmi.server.codebase="))
        rmiServerCodeBase = arg;
      else if (arg.startsWith("JAVA_HOME="))
        javaHome = arg.substring("JAVA_HOME=".length());
      else if (arg.startsWith("JAVA_MAX_HEAPSIZE="))
        strXmx = arg.substring("JAVA_MAX_HEAPSIZE=".length());
      else if (arg.startsWith("JAVA_MIN_HEAPSIZE="))
        strXms = arg.substring("JAVA_MIN_HEAPSIZE=".length());
      else if ((arg.indexOf("/FILE_SERVER=") == -1) && (arg.indexOf("/FILE_SERVER_PORT=") == -1) && (arg.indexOf("/CLIENT_DIR=") == -1) && (arg.indexOf("/MXDOC_HTTP_PROXY_HOST=") == -1) && (arg.indexOf("/MXDOC_HTTP_PROXY_PORT=") == -1) && (arg.indexOf("-Djava") == -1))
      {
        commandArgs = commandArgs + " " + arg;
      }
    }

    String javaCmd = "java ";
    if (strXmx != null) {
      javaCmd = javaCmd + "-Xmx" + strXmx + " ";
    }
    if (strXms != null) {
      javaCmd = javaCmd + "-Xms" + strXms + " ";
    }
    javaCmd = javaCmd + "-cp mxjboot.jar " + javaSecPolicy + " " + rmiServerCodeBase + " murex.rmi.loader.RmiLoader" + commandArgs;

    String[] cmdFile = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION", "REM Please do not modify this file.", "", "set JAVAHOME=" + javaHome, "set PATH=%JAVAHOME%\\jre\\bin;%JAVAHOME%\\jre\\bin\\client;%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", javaCmd, "", "endlocal" };

    writeToFile(cmdFile, webclientCmd);

    return webclientCmd;
  }

  public void createMxDocCmdFile(String[] args) throws Exception {
    String mxDocServerHost = getArgument(args, "/MXDOC_SERVER=");
    String mxDocServerPort = getArgument(args, "/MXDOC_SERVER_PORT=");
    String mxDocHttpProxyHost = getArgument(args, "/MXDOC_HTTP_PROXY_HOST=");
    String mxDocHttpProxyPort = getArgument(args, "/MXDOC_HTTP_PROXY_PORT=");
    String mxDocProxyAuthentificationName = getArgument(args, "/MXDOC_PROXY_AUTHENTIFICATION_NAME=");
    String mxDocProxyAuthentificationPass = getArgument(args, "/MXDOC_PROXY_AUTHENTIFICATION_PASSWD=");
    String jnlpClientPath = getArgument(args, "/CLIENT_DIR=");
    String destDir = computeClientDirPath(jnlpClientPath).concat("\\mxdoc");
    File jmdbrowserCmd = new File(destDir, "jmdbrowser.cmd");
    String javaHome = System.getProperty("java.home");
    String javaSecPolicy = "";
    String rmiServerCodeBase = "";

    for (int i = 0; i < args.length; i++) {
      String arg = args[i];
      if (arg.startsWith("-Djava.security.policy=")) {
        javaSecPolicy = arg;
      }
      if (arg.startsWith("-Djava.rmi.server.codebase=")) {
        rmiServerCodeBase = arg;
      }
    }

    String[] cmdFileNoProxy = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION.", "", "REM Do not remove the following line", "cd mxdoc", "", "set JAVAHOME=" + javaHome, "REM  (US character code)", "chcp 437", "set PATH=%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", "REM No HTTP Proxy used", "REM java -Xms256M -Xmx512M -Xss16M -cp ...", "java -cp mxjboot.jar " + javaSecPolicy + " -Djava.rmi.server.codebase=http://" + mxDocServerHost + ":" + mxDocServerPort + "/murex.download.jmdbrowser.download murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.JMDBrowser.JMDBrowser -fs:" + mxDocServerHost + ":" + mxDocServerPort + " %1 %2 %3 %4 %5 %6", "", "endlocal" };

    String[] cmdFileProxy = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION.", "", "REM Do not remove the following line", "cd mxdoc", "", "set JAVAHOME=" + javaHome, "REM  (US character code)", "chcp 437", "set PATH=%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", "REM HTTP Proxy used without authentification", "java -Dhttp.proxyHost=" + mxDocHttpProxyHost + " -Dhttp.proxyPort=" + mxDocHttpProxyPort + " -cp mxjboot.jar -Djava.security.policy=" + javaSecPolicy + " -Djava.rmi.server.codebase=http://" + mxDocServerHost + ":" + mxDocServerPort + "/murex.download.jmdbrowser.download murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.JMDBrowser.JMDBrowser -fs:" + mxDocServerHost + ":" + mxDocServerPort + " %1 %2 %3 %4 %5 %6", "", "endlocal" };

    String[] cmdFileProxyAuthentification = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION.", "", "REM Do not remove the following line", "cd mxdoc", "", "set JAVAHOME=" + javaHome, "REM  (US character code)", "chcp 437", "set PATH=%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", "REM HTTP Proxy used with authentification", "java -Dhttp.proxyHost=" + mxDocHttpProxyHost + " -Dhttp.proxyPort=" + mxDocHttpProxyPort + " -Dmurex.proxy.username=" + mxDocProxyAuthentificationName + " -Dmurex.proxy.password=" + mxDocProxyAuthentificationPass + " -cp mxjboot.jar -Djava.security.policy=" + javaSecPolicy + " -Djava.rmi.server.codebase=http://" + mxDocServerHost + ":" + mxDocServerPort + "/murex.download.jmdbrowser.download murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.JMDBrowser.JMDBrowser -fs:" + mxDocServerHost + ":" + mxDocServerPort + " %1 %2 %3 %4 %5 %6", "", "endlocal" };

    if (mxDocHttpProxyHost.equals("%MXDOC_HTTP_PROXY_HOST_NAME%")) {
      writeToFile(cmdFileNoProxy, jmdbrowserCmd);
    }
    else if (mxDocProxyAuthentificationName.equals("%MXDOC_PROXY_AUTHENTIFICATION_NAME%")) {
      writeToFile(cmdFileProxy, jmdbrowserCmd);
    }
    else
      writeToFile(cmdFileProxyAuthentification, jmdbrowserCmd);
  }

  private static void writeToFile(String[] lines, File file)
  {
    System.out.println("Build " + file.getAbsolutePath());

    if (file.exists()) {
      file.delete();
    }
    try
    {
      pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
    }
    catch (IOException e)
    {
      PrintWriter pw;
      throw new RuntimeException("Failed to create writer to file: " + file);
    }
    try
    {
      for (int i = 0; i < lines.length; i++)
        pw.println(lines[i]);
      pw.flush();
    }
    finally
    {
      PrintWriter pw;
      pw.close();
    }
  }

  void launchWebClient(File webClientCmdFile)
  {
    File workingDir = webClientCmdFile.getParentFile();
    try
    {
      System.out.println("Exec: " + webClientCmdFile + ", in working dir: " + workingDir);
      xProcess = Runtime.getRuntime().exec(webClientCmdFile.getAbsolutePath(), null, workingDir); } catch (IOException e) { Process xProcess;
      System.err.println("Failure executing process: " + webClientCmdFile.getAbsolutePath() + ", in working dir: " + workingDir);
      e.printStackTrace(System.err);
      return; }
    Process xProcess;
    File logFile = new File(workingDir, "jws.log");
    System.out.println("Logging process output to file: " + logFile);
    PrintWriter logFileWriter;
    try { PrintWriter logFileWriter = new PrintWriter(new FileOutputStream(logFile));

      ReaderWriterPipe stdoutReaderWriter = new ReaderWriterPipe(new BufferedReader(new InputStreamReader(xProcess.getInputStream())), logFileWriter);
      ReaderWriterPipe stderrReaderWriter = new ReaderWriterPipe(new BufferedReader(new InputStreamReader(xProcess.getErrorStream())), logFileWriter);

      new Thread(stdoutReaderWriter, "stdoutReader").start();
      new Thread(stderrReaderWriter, "stderrReader").start();
    } catch (FileNotFoundException e) {
      System.err.println("Failed to write to log file: " + logFile);
      e.printStackTrace(System.err);
      logFileWriter = null;
    }
    try
    {
      xProcess.waitFor();
    } catch (Exception e) {
      System.err.println(e.getMessage());
      e.printStackTrace(System.err);
    } finally {
      if (xProcess != null) {
        xProcess.destroy();
        xProcess = null;
      }
      if (logFileWriter != null) {
        logFileWriter.close();
        logFileWriter = null;
      }
      System.exit(0);
    }
  }

  private static final class ReaderWriterPipe
    implements Runnable
  {
    private BufferedReader reader;
    private PrintWriter pw;

    public ReaderWriterPipe(BufferedReader reader, PrintWriter pw)
    {
      this.reader = reader;
      this.pw = pw;
    }

    public void run()
    {
      try {
        while ((this.reader != null) && (this.pw != null)) {
          String sLine = this.reader.readLine();
          if (sLine == null) {
            break;
          }
          System.out.println(sLine);
          this.pw.println(sLine);
          this.pw.flush();
        }
      } catch (IOException e) {
        e.printStackTrace(System.err);
      } finally {
        if (this.reader != null) {
          try {
            this.reader.close();
          } catch (IOException e) {
            e.printStackTrace(System.err);
          }
          this.reader = null;
        }

        if (this.pw != null) {
          this.pw.close();
          this.pw = null;
        }
      }
    }
  }
}

package murex.rmi.loader;

import java.io.DataInputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintStream;
import java.net.URL;
import java.security.Permission;
import java.security.SecureClassLoader;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.Hashtable;
import java.util.Map;
import java.util.Properties;
import java.util.Vector;
import java.util.jar.JarFile;
import java.util.zip.ZipEntry;
import murex.rmi.loader.parser.CheckFile;
import murex.rmi.loader.parser.FileVersion;
import murex.rmi.loader.parser.Parser;
import murex.rmi.loader.parser.VersionParser;

public class FileServerClassLoader extends SecureClassLoader
{
  public static final String strJarFolder = "jar/";
  public static final String strBinFolder = "bin/";
  public static final String strFileVersion = "file.version";
  public static final String tagFiles = "Files";
  public static final String tagFile = "File";
  public static final String tagVersion = "Version";
  public static final String tagCheckSum = "CheckSum";
  private JarFile[] arrayJarFiles;
  private Hashtable hashtableMemory;

  FileServerClassLoader(URL url, URL urlFileServerSite)
    throws Exception
  {
    super(FileServerClassLoader.class.getClassLoader());
    String fileClassPath = FileServerGenericLoader.fileServletDownload();
    if (fileClassPath == null) {
      if ((url.getFile().trim().indexOf(".download") == -1) && (url.getFile().trim().indexOf(".classpath") == -1))
        throw new Exception(" no file .download is specified in your codebase.");
      fileClassPath = url.getFile();
    }

    String strAdaptedFilePath = fileClassPath;
    int iLastSeparatorIndex = fileClassPath.lastIndexOf(".");
    if (iLastSeparatorIndex != -1) {
      String strFilePathToAdapt = fileClassPath.substring(0, iLastSeparatorIndex);
      strFilePathToAdapt = strFilePathToAdapt.replace('.', '/');
      strAdaptedFilePath = strFilePathToAdapt + fileClassPath.substring(iLastSeparatorIndex);
    }
    fileClassPath = strAdaptedFilePath;

    createSubDirectories();

    byte[] jarListFile = FileServerGenericLoader.getFileFromFileServer(url, fileClassPath);
    String strJarListFile = new String(jarListFile);
    jarListFile = null;

    Parser parser = new Parser(strJarListFile);
    strJarListFile = null;
    CheckFile[] physical = parser.getPhysicalDownload();
    CheckFile[] memory = parser.getMemoryDownload();
    parser = null;

    if (RmiLoader.getMxProperties().get("murex.application.jni") == null)
    {
      URL physicalURL;
      URL physicalURL;
      if (urlFileServerSite != null)
        physicalURL = urlFileServerSite;
      else {
        physicalURL = url;
      }

      String strBinFileVersion = "bin/file.version";
      File fBinFileVersion = new File(strBinFileVersion);
      StringBuffer binStringBuffer = null;
      Map binFileVersion = null;
      if (fBinFileVersion.exists()) {
        System.out.println("Opening " + strBinFileVersion + "...");
        try {
          VersionParser versionParser = new VersionParser(fBinFileVersion.getPath());
          binFileVersion = versionParser.getFileVersion();
        } catch (Exception e) {
          e.printStackTrace();
        }
      }

      String strJarFileVersion = "jar/file.version";
      File fJarFileVersion = new File(strJarFileVersion);
      StringBuffer jarStringBuffer = null;
      Map jarFileVersion = null;
      if (fJarFileVersion.exists()) {
        System.out.println("Opening " + strJarFileVersion + "...");
        try {
          VersionParser versionParser = new VersionParser(fJarFileVersion.getPath());
          jarFileVersion = versionParser.getFileVersion();
        } catch (Exception e) {
          e.printStackTrace();
        }
      }

      File fLocalFileVersion = new File("file.version");
      StringBuffer localStringBuffer = null;
      Map localFileVersion = null;
      if (fLocalFileVersion.exists()) {
        System.out.println("Opening file.version...");
        try {
          VersionParser versionParser = new VersionParser(fLocalFileVersion.getPath());
          localFileVersion = versionParser.getFileVersion();
        } catch (Exception e) {
          e.printStackTrace();
        }

      }

      boolean bJFVdownload = false;
      boolean bBFVdownload = false;
      boolean bLFVdownload = false;

      System.out.println("Checking files...");
      ArrayList arrayJar = new ArrayList(physical.length);
      StringBuffer strCodebase = new StringBuffer("");
      for (int j = 0; j < physical.length; j++) {
        byte[] jarFile = null;
        String strFile = physical[j].getFile();
        String strDestFile = getDestFileName(strFile);
        boolean bGet = true;
        String strCheck = physical[j].getCheck();
        String strCheckSum = null;
        String strVersion = null;

        if ((strCheck != null) && (strCheck.equals("N"))) {
          bGet = !new File(strDestFile).exists();
          if (bGet) {
            strCheckSum = physical[j].getCheckSum();
          }
          else
          {
            Map map;
            Map map;
            if (isInJarFolder(strFile)) {
              map = jarFileVersion;
            }
            else
            {
              Map map;
              if (isInBinFolder(strFile))
                map = binFileVersion;
              else {
                map = localFileVersion;
              }
            }
            if (map != null) {
              FileVersion fv = (FileVersion)map.get(strFile);
              if ((fv != null) && 
                (fv.getCheckSum() != null)) {
                strCheckSum = fv.getCheckSum();
              }
            }

            if (strCheckSum == null) {
              strCheckSum = FileServerGenericLoader.getCheckSum(strFile);
            }
          }
        }
        else if ((strCheck != null) && (strCheck.equals("Y"))) {
          strCheckSum = physical[j].getCheckSum();
          if (!new File(strDestFile).exists()) {
            bGet = true;
          }
          else if (strCheckSum != null)
          {
            Map map;
            Map map;
            if (isInJarFolder(strFile)) {
              map = jarFileVersion;
            }
            else
            {
              Map map;
              if (isInBinFolder(strFile))
                map = binFileVersion;
              else {
                map = localFileVersion;
              }
            }
            String strCheckSum2 = null;
            if (map != null) {
              FileVersion fv = (FileVersion)map.get(strFile);
              if ((fv != null) && 
                (fv.getCheckSum() != null)) {
                strCheckSum2 = fv.getCheckSum();
              }
            }

            if (strCheckSum2 == null) {
              strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
            }
            bGet = !strCheckSum2.equals(strCheckSum);
          }

        }
        else if ((strCheck != null) && (strCheck.equals("A"))) {
          if (physical[j].getVersion() != null) {
            strVersion = physical[j].getVersion();
          }
          if (physical[j].getCheckSum() != null) {
            strCheckSum = physical[j].getCheckSum();
          }

          if (!new File(strDestFile).exists()) {
            bGet = true;
          }
          else
          {
            Map map;
            Map map;
            if (isInJarFolder(strFile)) {
              map = jarFileVersion;
            }
            else
            {
              Map map;
              if (isInBinFolder(strFile))
                map = binFileVersion;
              else {
                map = localFileVersion;
              }
            }
            if (map != null) {
              FileVersion fv = (FileVersion)map.get(strFile);
              if (fv != null) {
                if ((strVersion != null) && (fv.getVersion() != null)) {
                  bGet = !fv.getVersion().equals(strVersion);
                }
                else if (strCheckSum != null) {
                  if (fv.getCheckSum() != null) {
                    bGet = !fv.getCheckSum().equals(strCheckSum);
                  } else {
                    String strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
                    bGet = !strCheckSum2.equals(strCheckSum);
                  }
                }
              }
              else if (strCheckSum != null) {
                String strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
                bGet = !strCheckSum2.equals(strCheckSum);
              }

            }
            else if (strCheckSum != null) {
              String strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
              bGet = !strCheckSum2.equals(strCheckSum);
            }
          }

        }

        if (bGet)
          try {
            jarFile = FileServerGenericLoader.getFileFromFileServer(physicalURL, strFile);
          } catch (Exception err1) {
            err1.printStackTrace();
            jarFile = null;
          }
        else {
          jarFile = new byte[0];
        }

        if (jarFile != null)
        {
          if (jarFile.length != 0) {
            FileOutputStream jarFileOutputStream = new FileOutputStream(strDestFile);
            jarFileOutputStream.write(jarFile);
            jarFileOutputStream.close();
            System.out.println(strDestFile + " copied.");
          }
          StringBuffer sb;
          StringBuffer sb;
          if (isInJarFolder(strFile)) {
            if (bGet)
            {
              bJFVdownload = true;
            }
            arrayJar.add(new JarFile(strDestFile));

            strCodebase.append("file:./");
            strCodebase.append(strDestFile);
            strCodebase.append(" ");

            if (jarStringBuffer == null) {
              jarStringBuffer = new StringBuffer();
              jarStringBuffer.append("<"); jarStringBuffer.append("Files"); jarStringBuffer.append(">\r\n");
            }
            sb = jarStringBuffer;
          }
          else
          {
            StringBuffer sb;
            if (isInBinFolder(strFile)) {
              if (bGet) {
                bBFVdownload = true;
              }
              if (binStringBuffer == null) {
                binStringBuffer = new StringBuffer();
                binStringBuffer.append("<"); binStringBuffer.append("Files"); binStringBuffer.append(">\r\n");
              }
              sb = binStringBuffer;
            }
            else {
              if (bGet)
              {
                bLFVdownload = true;
              }
              if (localStringBuffer == null) {
                localStringBuffer = new StringBuffer();
                localStringBuffer.append("<"); localStringBuffer.append("Files"); localStringBuffer.append(">\r\n");
              }
              sb = localStringBuffer;
            }
          }
          sb.append("<"); sb.append("File");
          if (strVersion != null) {
            sb.append(" ");
            sb.append("Version"); sb.append("=\""); sb.append(strVersion); sb.append("\"");
          }
          if (strCheckSum != null) {
            sb.append(" ");
            sb.append("CheckSum"); sb.append("=\""); sb.append(strCheckSum); sb.append("\"");
          }
          sb.append(">");
          sb.append(strFile);
          sb.append("</"); sb.append("File"); sb.append(">\r\n");
        }
      }
      if (!fBinFileVersion.exists()) {
        bBFVdownload = true;
      }
      if (!fJarFileVersion.exists()) {
        bJFVdownload = true;
      }
      if (!fLocalFileVersion.exists()) {
        bLFVdownload = true;
      }

      if ((binStringBuffer != null) && (bBFVdownload)) {
        binStringBuffer.append("</"); binStringBuffer.append("Files"); binStringBuffer.append(">\r\n");
        try {
          if (fBinFileVersion.exists()) {
            fBinFileVersion.delete();
          }
          FileOutputStream jarFileOutputStream = new FileOutputStream(strBinFileVersion);
          jarFileOutputStream.write(binStringBuffer.toString().getBytes());
          jarFileOutputStream.close();
          System.out.println(strBinFileVersion + " saved.");
        } catch (Exception e) {
          System.out.println(e.getMessage());
        }
      }

      if ((jarStringBuffer != null) && (bJFVdownload)) {
        jarStringBuffer.append("</"); jarStringBuffer.append("Files"); jarStringBuffer.append(">\r\n");
        try {
          if (fJarFileVersion.exists()) {
            fJarFileVersion.delete();
          }
          FileOutputStream jarFileOutputStream = new FileOutputStream(strJarFileVersion);
          jarFileOutputStream.write(jarStringBuffer.toString().getBytes());
          jarFileOutputStream.close();
          System.out.println(strJarFileVersion + " saved.");
        } catch (Exception e) {
          System.out.println(e.getMessage());
        }
      }

      if ((localStringBuffer != null) && (bLFVdownload)) {
        localStringBuffer.append("</"); localStringBuffer.append("Files"); localStringBuffer.append(">\r\n");
        try {
          if (fLocalFileVersion.exists()) {
            fLocalFileVersion.delete();
          }
          FileOutputStream jarFileOutputStream = new FileOutputStream("file.version");
          jarFileOutputStream.write(localStringBuffer.toString().getBytes());
          jarFileOutputStream.close();
          System.out.println("file.version saved.");
        } catch (Exception e) {
          System.out.println(e.getMessage());
        }
      }

      this.arrayJarFiles = ((JarFile[])arrayJar.toArray(new JarFile[0]));

      System.getProperties().put("java.rmi.server.codebase", strCodebase.toString());
      this.hashtableMemory = new Hashtable();
      for (int j = 0; j < memory.length; j++) {
        String strFile = memory[j].getFile();
        byte[] classFile = null;
        try {
          classFile = FileServerGenericLoader.getFileFromFileServer(url, strFile);
        } catch (Exception err1) {
          classFile = null;
        }
        if (classFile != null)
          this.hashtableMemory.put(strFile, classFile);
      }
    }
    else {
      StringBuffer strCodebase = new StringBuffer("");
      JarFile[] jarFile = new JarFile[physical.length];
      int iCount = 0;
      for (int j = 0; j < physical.length; j++)
        try {
          String strFile = physical[j].getFile();
          if (isInJarFolder(strFile)) {
            String strDestFile = getDestFileName(strFile);
            jarFile[j] = new JarFile(strDestFile);
            iCount++;
            strCodebase.append("file:./");
            strCodebase.append(strDestFile);
            strCodebase.append(" ");
          }
        }
        catch (Exception e) {
        }
      System.getProperties().put("java.rmi.server.codebase", strCodebase.toString());

      this.arrayJarFiles = new JarFile[iCount];
      int j = 0;
      for (int i = 0; i < jarFile.length; i++) {
        if (jarFile[i] != null) {
          this.arrayJarFiles[j] = jarFile[i];
          j++;
        }
      }
      jarFile = null;
      strCodebase = null;

      this.hashtableMemory = new Hashtable();
      for (j = 0; j < memory.length; j++) {
        String strFile = memory[j].getFile();
        byte[] classFile = null;
        try {
          classFile = FileServerGenericLoader.getFileFromFileServer(url, strFile);
        } catch (Exception err1) {
          classFile = null;
        }
        if (classFile != null)
          this.hashtableMemory.put(strFile, classFile);
      }
    }
    physical = null;
    memory = null;
  }

  public Class findClass(String name) throws ClassNotFoundException {
    Class classData = null;
    byte[] b = loadClassData(name);
    if (b == null)
      throw new ClassNotFoundException(name);
    int index = name.lastIndexOf('.');
    if (index != -1) {
      String pkgname = name.substring(0, index);
      Package pkg = getPackage(pkgname);
      if (pkg == null) {
        definePackage(pkgname, null, null, null, null, null, null, null);
      }
    }
    classData = defineClass(name, b, 0, b.length);
    return classData;
  }

  private byte[] loadClassData(String name) {
    byte[] bytecodes = null;
    name = name.replace('.', '/') + ".class";
    for (int i = 0; (i < this.arrayJarFiles.length) && (bytecodes == null); i++)
      try {
        ZipEntry zipEntry = this.arrayJarFiles[i].getEntry(name);
        if (zipEntry != null) {
          InputStream in1 = this.arrayJarFiles[i].getInputStream(zipEntry);
          DataInputStream in2 = new DataInputStream(in1);
          bytecodes = new byte[(int)zipEntry.getSize()];
          in2.readFully(bytecodes);
          try {
            in2.close();
          } catch (Exception ee) {
          }
        }
      }
      catch (Exception e) {
      }
    if (bytecodes == null) {
      bytecodes = (byte[])this.hashtableMemory.get(name);
    }
    return bytecodes;
  }

  protected Enumeration findResources(String name)
    throws IOException
  {
    Vector v = null;
    for (int i = 0; i < this.arrayJarFiles.length; i++) {
      try {
        ZipEntry zipEntry = this.arrayJarFiles[i].getEntry(name);
        if (zipEntry != null) {
          if (v == null) {
            v = new Vector();
          }
          v.add(new URL("jar:file:./" + getDestFileName(this.arrayJarFiles[i].getName()) + "!/" + name));
        }
      } catch (Exception e) {
        if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
          e.printStackTrace();
        }
      }
    }
    if (v != null) {
      return v.elements();
    }
    return null;
  }

  protected URL findResource(String name)
  {
    URL url = null;
    for (int i = 0; (i < this.arrayJarFiles.length) && (url == null); i++) {
      try {
        ZipEntry zipEntry = this.arrayJarFiles[i].getEntry(name);
        if (zipEntry != null)
          url = new URL("jar:file:./" + getDestFileName(this.arrayJarFiles[i].getName()) + "!/" + name);
      }
      catch (Exception e) {
        if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
          e.printStackTrace();
        }
      }
    }
    return url;
  }

  public static String getDestFileName(String strFile)
  {
    String strDestFile;
    String strDestFile;
    if (isInBinFolder(strFile)) {
      strDestFile = "bin/" + getFileName(strFile);
    }
    else
    {
      String strDestFile;
      if (isInJarFolder(strFile))
        strDestFile = "jar/" + getFileName(strFile);
      else
        strDestFile = getFileName(strFile);
    }
    return strDestFile;
  }

  public static boolean isInJarFolder(String strFile) {
    return strFile.endsWith("jar");
  }

  public static boolean isInBinFolder(String strFile) {
    return (strFile.endsWith(".dll")) || (strFile.endsWith(".so")) || (strFile.endsWith(".a")) || (strFile.endsWith(".sl")) || (strFile.endsWith(".exe")) || (strFile.endsWith(".tlb"));
  }

  private static void createSubDirectories() {
    File fileJarFolder = new File("jar/");
    if (!fileJarFolder.exists()) {
      if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
        System.out.println("Creating folder jar/");
      }
      fileJarFolder.mkdir();
    }

    File fileBinFolder = new File("bin/");
    if (!fileBinFolder.exists()) {
      if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
        System.out.println("Creating folder bin/");
      }
      fileBinFolder.mkdir();
    }
  }

  private static String getFileName(String strPath) {
    String strFileName = strPath;
    int iSeparatorIndex = strPath.lastIndexOf("/");
    if (iSeparatorIndex != -1) {
      strFileName = strPath.substring(iSeparatorIndex + 1);
    }
    iSeparatorIndex = strPath.lastIndexOf("\\");
    if (iSeparatorIndex != -1) {
      strFileName = strPath.substring(iSeparatorIndex + 1);
    }
    return strFileName;
  }

  private class ClassLoaderSecurityManager extends SecurityManager
  {
    private ClassLoaderSecurityManager()
    {
    }

    public void checkPermission(Permission perm)
    {
    }

    public void checkPermission(Permission perm, Object context)
    {
    }
  }
}

package murex.rmi.loader;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.Authenticator;
import java.net.PasswordAuthentication;
import java.net.URL;
import java.net.URLConnection;
import java.util.Enumeration;
import java.util.Hashtable;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

public class FileServerGenericLoader
{
  public static final String tagFile = "File";
  public static final String tagVersion = "Version";
  public static final String tagCheckSum = "CheckSum";
  public static final String tagException = "EXCEPTION";
  public static final String tagMurexAnswer = "Murex-Answer";
  public static final int nbFile = 30;

  public static byte[] getFileFromFileServer(URL url, String strFile, boolean doCheck, String strVersion, String strCheckSum)
    throws Exception
  {
    if (strFile.charAt(0) != '/') {
      strFile = "/" + strFile;
    }

    String strParameter = "";
    if (doCheck) {
      if (strVersion != null) {
        strParameter = "Version=" + strVersion;
      }
      if (strCheckSum != null) {
        if (strVersion != null) {
          strParameter = strParameter + "&";
        }
        strParameter = strParameter + "CheckSum=" + strCheckSum;
      }

      if ((strVersion == null) && (strCheckSum == null)) {
        strCheckSum = getCheckSum(strFile);
        if (strCheckSum != null) {
          strParameter = "CheckSum=" + strCheckSum;
        }
      }
    }

    if (!strParameter.equals("")) {
      strParameter = "&" + strParameter;
    }
    strParameter = "InternalVersion=1" + strParameter;
    String strRequest;
    if (fileServletDownload() != null) {
      String strRequest = url.getFile();
      strRequest = strRequest + "?File=" + strFile;
      if (!strParameter.equals(""))
        strRequest = strRequest + "&" + strParameter;
    }
    else {
      strRequest = strFile;
      if (!strParameter.equals("")) {
        strRequest = strRequest + "?" + strParameter;
      }
    }

    URL urlToFileServer = new URL(url.getProtocol(), url.getHost(), url.getPort(), strRequest);
    URLConnection urlConnection = urlToFileServer.openConnection();
    urlConnection.setUseCaches(false);
    urlConnection.setRequestProperty("MUREX-EXTENSION", "murex/murex.rmi.loader.FileServerGenericLoader");
    InputStream inputStream = urlConnection.getInputStream();

    if (urlConnection.getContentLength() == -1) {
      throw new Exception("unable to get file " + strFile + " from " + url + ".");
    }

    String strStatus = urlConnection.getHeaderField("Murex-Answer");
    if ((strStatus != null) && (strStatus.equals("EXCEPTION"))) {
      byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
      throw new Exception(new String(inputFile));
    }

    byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
    inputStream.close();
    return inputFile;
  }

  public static byte[] getFileFromFileServer(URL url, String strFile) throws Exception {
    return getFileFromFileServer(url, strFile, false, null, null);
  }

  public static String fileServletDownload() {
    return (String)RmiLoader.getMxProperties().get("/MXJ_SERVLET_DOWNLOAD:");
  }

  public static String getCheckSum(String strFile) {
    String strCheckSum = null;
    if (FileServerClassLoader.isInJarFolder(strFile))
      try {
        String strDestFile = FileServerClassLoader.getDestFileName(strFile);
        JarFile jarFile = new JarFile(strDestFile);
        Enumeration enum = jarFile.entries();
        StringBuffer result = new StringBuffer();
        int nb = 0;
        long sum = 0L;
        while (enum.hasMoreElements()) {
          JarEntry jarEntry = (JarEntry)enum.nextElement();
          if (nb < 30) {
            sum += jarEntry.getSize();
          } else {
            result.append(sum);
            sum = jarEntry.getSize();
            nb = 0;
          }
        }
        if (sum != 0L) {
          result.append(sum);
        }
        jarFile.close();
        strCheckSum = result.toString();
      }
      catch (Exception e) {
      }
    else try {
        File file = new File(FileServerClassLoader.getDestFileName(strFile));
        strCheckSum = file.length() + "";
      }
      catch (Exception e)
      {
      } return strCheckSum;
  }

  public static byte[] getBodyStream(InputStream inputStream, int count) {
    byte[] tab = new byte[count];
    try {
      int iReceived = 0;
      int iTempReceived = 0;
      while ((iTempReceived = inputStream.read(tab, iReceived, count - iReceived)) > 0)
        iReceived += iTempReceived;
    }
    catch (IOException e) {
    }
    return tab;
  }

  static
  {
    if ((System.getProperty("murex.proxy.username") != null) && (System.getProperty("murex.proxy.password") != null))
      Authenticator.setDefault(new PasswordAuthenticator(System.getProperty("murex.proxy.username"), System.getProperty("murex.proxy.password"))); 
  }

  private static class PasswordAuthenticator extends Authenticator {
    private String strUsername;
    private String strPassword;
    private PasswordAuthentication passwordAuthentication;

    public PasswordAuthenticator(String strUsername, String strPassword) { this.passwordAuthentication = new PasswordAuthentication(strUsername, strPassword.toCharArray()); }

    protected PasswordAuthentication getPasswordAuthentication()
    {
      return this.passwordAuthentication;
    }
  }
}

package murex.rmi.loader;

import java.net.URL;
import java.rmi.RMISecurityManager;
import java.security.Permission;
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Map;
import java.util.Properties;
import java.util.StringTokenizer;

public class RmiLoader
{
  public static final String MXJ_ARGS_CLASS_NAME = "/MXJ_CLASS_NAME:";
  public static final String MXJ_ARGS_FILE_SERVER_REMOTE_SITE_URL = "/MXJ_FILE_SERVER_REMOTE_SITE_URL:";
  public static final String propertyFILE_SERVER_REMOTE_SITE_URL = "murex.application.codebase.remote.site";
  public static final String MXJ_ARGS_SERVLET_DOWNLOAD = "/MXJ_SERVLET_DOWNLOAD:";
  protected static Hashtable mxProperties = new Hashtable();
  protected URL urlJarFile;
  protected URL urlFileServerSite;
  protected String strClientClassName;
  protected FileServerClassLoader fileServerClassLoader;

  protected RmiLoader(String strURL, String strClientClassName)
    throws Exception
  {
    System.setSecurityManager(new RMIClientBootstrapSecurityManager(null));
    this.strClientClassName = strClientClassName;
    int index = strURL.indexOf("?");
    if (index != -1) {
      Map mapParameter = new HashMap();
      String strQuery = strURL.substring(index + 1);
      StringTokenizer stk = new StringTokenizer(strQuery, "&");
      while (stk.hasMoreElements()) {
        String str = (String)stk.nextElement();
        String strName = str.substring(0, str.indexOf("="));
        String strValue = str.substring(str.indexOf("=") + 1);
        mapParameter.put(strName.trim(), strValue.trim());
      }
      strURL = strURL.substring(0, index);
      mxProperties.put("/MXJ_SERVLET_DOWNLOAD:", mapParameter.get("File"));
    }
    initURL(strURL);
  }

  protected RmiLoader(String[] strArgs) throws Exception {
    System.setSecurityManager(new RMIClientBootstrapSecurityManager(null));
    for (int i = 0; i < strArgs.length; i++) {
      if (strArgs[i].startsWith("/MXJ_CLASS_NAME:"))
        this.strClientClassName = strArgs[i].substring("/MXJ_CLASS_NAME:".length());
      else if (strArgs[i].startsWith("/MXJ_FILE_SERVER_REMOTE_SITE_URL:"))
        this.urlFileServerSite = new URL(strArgs[i].substring("/MXJ_FILE_SERVER_REMOTE_SITE_URL:".length()));
      else if (strArgs[i].startsWith("/MXJ_SERVLET_DOWNLOAD:")) {
        mxProperties.put("/MXJ_SERVLET_DOWNLOAD:", strArgs[i].substring("/MXJ_SERVLET_DOWNLOAD:".length()));
      }
    }
    mxProperties.put("murex.rmi.loader.RmiLoader.arguments", strArgs);
    Properties p = System.getProperties();
    String strURL = p.getProperty("murex.application.codebase") == null ? p.getProperty("java.rmi.server.codebase") : p.getProperty("murex.application.codebase");
    initURL(strURL);
  }

  private void initURL(String strURL) throws Exception {
    if (strURL == null)
      throw new Exception(" no codebase specified.");
    mxProperties.put("murex.application.codebase", strURL);
    this.urlJarFile = new URL(strURL);
  }

  public static Hashtable getMxProperties() {
    return mxProperties;
  }

  public Object startRunning() throws Exception {
    if ((this.strClientClassName == null) || (this.strClientClassName.equals(""))) {
      throw new Exception("Class name not found");
    }

    if ((mxProperties.get("/MXJ_FILE_SERVER_REMOTE_SITE_URL:") != null) && (this.urlFileServerSite == null)) {
      this.urlFileServerSite = new URL((String)mxProperties.remove("/MXJ_FILE_SERVER_REMOTE_SITE_URL:"));
    }

    if (this.urlFileServerSite == null) {
      Properties p = System.getProperties();
      String strFileServerSite = (String)p.get("murex.application.codebase.remote.site");
      if (strFileServerSite != null) {
        this.urlFileServerSite = new URL(strFileServerSite);
      }
    }
    if (this.urlFileServerSite != null) {
      mxProperties.put("murex.application.codebase.remote.site", this.urlFileServerSite.toString());
    }

    this.fileServerClassLoader = new FileServerClassLoader(this.urlJarFile, this.urlFileServerSite);
    Thread.currentThread().setContextClassLoader(this.fileServerClassLoader);
    Class clientClass = Class.forName(this.strClientClassName, true, this.fileServerClassLoader);

    Runnable client = (Runnable)clientClass.newInstance();
    client.run();
    return client;
  }

  public Class findClass(String strClassName) throws Exception {
    return this.fileServerClassLoader.loadClass(strClassName);
  }

  public static void main(String[] strArgs)
  {
    try
    {
      RmiLoader rmiLoader = new RmiLoader(strArgs);
      rmiLoader.startRunning();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  private class RMIClientBootstrapSecurityManager extends RMISecurityManager
  {
    private RMIClientBootstrapSecurityManager()
    {
    }

    public synchronized void checkCreateClassLoader()
    {
    }

    public synchronized void checkConnect(String host, int port)
    {
    }

    public synchronized void checkAccess(Thread t)
    {
    }

    public synchronized void checkAccess(ThreadGroup g)
    {
    }

    public synchronized void checkPropertiesAccess()
    {
    }

    public void checkPermission(Permission perm)
    {
    }

    public void checkPermission(Permission perm, Object context)
    {
    }

    RMIClientBootstrapSecurityManager(RmiLoader.1 x1)
    {
      this();
    }
  }
}

package murex.rmi.loader;

import java.io.PrintStream;

public class Version
{
  public static final String strVersion = "1";
  public static final String tagInternalVersion = "InternalVersion";

  public static void printVersion()
  {
    System.out.println("The version of this class loader is 1.");
    System.out.println("It downloads files which extensions are .download.");
    System.out.println("This kind of files contains the list of files to be downloaded with its checksums and versions (if declared in the fileserver).");
    System.out.println("It can download this file from a file server or from a servlet.");
    System.out.println("");
    System.out.println("The downloading of a given file is decided by checking the Check attribute as follows : ");
    System.out.println("");
    System.out.println("    1-The attribute Check is set to N (No) :");
    System.out.println("    -------------------------------------------");
    System.out.println("    if the file exists nothing is done, if not the file is downloaded.");
    System.out.println("");
    System.out.println("    2-The attribute Check is set to Y (Yes) :");
    System.out.println("    -------------------------------------------");
    System.out.println("    if the file exists and the checksum is equal to the one given by the fileserver nothing is done, if not the file is downloaded.");
    System.out.println("");
    System.out.println("    3-The attribute Check is set to A (Automatic) :");
    System.out.println("    -------------------------------------------");
    System.out.println("    if the file has a version and is the same than the one given by the fileserver nothing is done, if not step 2 is done.");
    System.out.println("");
    System.out.println("At the end, versions and checksums are saved locally in files file.version.");
  }

  public static void main(String[] argv) {
    printVersion();
  }
}

package murex.xml.server.mx.mxnative.mxloader;

import java.io.PrintStream;
import java.util.Hashtable;
import murex.rmi.loader.RmiLoader;

public class RmiMxLoader extends RmiLoader
{
  private static PrintStream printStream = null;

  protected RmiMxLoader(String strURL, String strClientClassName)
    throws Exception
  {
    super(strURL, strClientClassName);
  }

  public static RmiMxLoader create(String strURL) {
    RmiMxLoader rmiMxLoader = null;
    try {
      rmiMxLoader = new RmiMxLoader(strURL, "murex.xml.server.mx.mxnative.MxHome");
      mxProperties.put("murex.application.jni", "true");
    } catch (Exception e) {
      printStackTrace(e);
    }
    return rmiMxLoader;
  }

  public static RmiMxLoader create(String strURL, String strHomeClass) {
    RmiMxLoader rmiMxLoader = null;
    try {
      rmiMxLoader = new RmiMxLoader(strURL, strHomeClass);
    } catch (Exception e) {
      printStackTrace(e);
    }
    return rmiMxLoader;
  }

  public static RmiMxLoader create(String strURL, String strHomeClass, String strJniMode) {
    RmiMxLoader rmiMxLoader = null;
    try {
      rmiMxLoader = new RmiMxLoader(strURL, strHomeClass);
      if ((strJniMode != null) && (strJniMode.equals("yes")))
        mxProperties.put("murex.application.jni", "true");
    }
    catch (Exception e) {
      printStackTrace(e);
    }
    return rmiMxLoader;
  }

  public void pushArguments(String strCode, String strValue) {
    mxProperties.put(strValue, strCode);
  }

  public Object commitArguments() {
    Object obj = null;
    try {
      obj = startRunning();
    } catch (Exception e) {
      printStackTrace(e);
    }
    return obj;
  }

  public Class findClass(String strClassName) {
    Class c = null;
    try {
      c = super.findClass(strClassName);
    } catch (Exception e) {
      printStackTrace(e);
    }
    return c;
  }

  public static void printStackTrace(Throwable e)
  {
    if (printStream != null)
      e.printStackTrace(printStream);
  }

  public static void println(String str)
  {
    if (printStream != null)
      printStream.println(str);
  }
}


## All Murex 2.1 packages (from  clease case server) 
[1477277@dl1101 ~]$ bash
[1477277@dl1101 ~]$ cd /
[1477277@dl1101 /]$ ls
a     b         bin   c       clearcase  dev_vob      etc   lib    lost+found  mnt  opt  proc  rel_vob  sbin     shared  sys  usr  view
apps  bhanu.sh  boot  cgroup  dev        dev_vob_new  home  lib64  media       nsr  pfs  puma  root     selinux  srv     tmp  var  vobs
[1477277@dl1101 /]$ cd dev_vob
[1477277@dl1101 dev_vob]$ ls
common  packages
[1477277@dl1101 dev_vob]$ cd packages
[1477277@dl1101 packages]$ ls
blizzard  castor  enconnect    mars  pollux    preprocessor  scrittura2_new  scritturamis   vast2.0
borvo2    dcrm    leps_script  nike  poseidon  s2bfx         scrittura2_prj  syndicatebook  wmb
[1477277@dl1101 packages]$ cd poseidon/
[1477277@dl1101 poseidon]$ ls
[1477277@dl1101 poseidon]$ ct setview 1477277
[1477277@dl1101 poseidon]$ ls
bo               dm_audit_exts    dm_pl_ext       dpstools        housekeeping      medusa              mx_api               pandora         regression
cadex            dm_basel         dm_pl_fdr       ebbs            install-me.sh     merlin              mx_api_x86           pay_db_objects  sabre_ird_fxo
calbal           dm_cash_flow     dm_pmnt_fdr     ebs             irec              mktrates            mx_api_x86_var       pemtools        SolarisStudioX86
callcontra       dm_complvar_fdr  dmpv01          endymion        itrs              mls                 mxml                 perseus         tools
caxton           dm_com_sim       dm_reg_exts     eod             itrstools         model               mxtools              plutus          toyota
common           dm_cs_ext        dm_sentry       eod_auto        jobdefs           monitor             NAG                  plutus2         triton
common_x86       dm_cs_fdr        dm_sim_fdr      Extractions     launchers         murex               ndf                  pos_calrecon    tt.txt
crt              dm_fdr_sens      dm_static_exts  fabs            lost+found        murexg2000          nike                 pos_mxisft      varcon
data_conversion  dm_maketer_ext   dm_static_fdr   fdr_db_objects  Makefilebuild.sh  murex_housekeeping  objectsrelease_info  primetrade      varex
datamart         dm_mv_ext        dm_sv_fdr       Feeders         markit            murex_lib           old_code             prorisk         varutils
dm_acct_fdr      dm_mv_fdr        dm_test         fm_tools        mats              murfi               olink                proteus         web
[1477277@dl1101 poseidon]$


## application SERVER
						
在B/S模式中，服务器端用servlet来为web客户的请求提供服务。Servlet服务扩展了web服务器的功能，即由web服务器接收和处理客户请求，然后把请求传给servlet,并把servlet处理的结果返回给客户。						
servlet 容器与servlet之间的接口是由java.servlet.api定义的，在此api中定义了servlet的各种方法，这些方法在servlet生命周期的不同阶段被servlet容器调用，tomcat服务器由一系列可配置的组件构成，tomcat组件可以在conf/server.xml文件当中进行配置。组件包括：						
						
<Server> => 代表整个servelet容器，是tomcat实例的顶层元素。						
      <Service>						
			<Connector>			
			     客户与服务器之间的通信接口			
			</Connector>			
			<Connector>			
			</Connector>			
			<Engine> => 处理在同一个<Service>中所有<Connector>元素接受到的用户请求			
				<Host>  => 每个<Host>元素定义了一个虚拟主机，它可以包含一个或多个web应用。		
					<Context path=''> 	
					     为特定的web应用处理所有用户请求，每个<Context>元素代表了运行在虚拟机上的单个web应用。	
						 每个web应用有唯一的Context，当java web应用运行时，Servlet容器为每个web应用创建唯一的ServletContext对象，它被整个web应用中所有的组件共享
					</Context>	
					...	
					<Context> 	
					</Context>	
				</Host>		
			</Engine>			
      </Service>						
	  					
	  <Service>					
	  </Service>					
</Server>						
						
						
Murex采用C/S结构实现的，所以Server端没有Tomcat服务器，需要我们自己用TCP/IP socket来实现类似于TOMCAT的功能。						
类似一个应用服务器中可以部署多个应用程序，我们的服务器端也可以配置和部署多个Murex应用，以为团队提供多个测试环境。						
						
Murex Clinet 先通过RMI技术发送请求给fileServer as All Resource Files all are deployed on file server.. 						
Once File server received request it will load sites.mxres (./fs/public/mxres/sites/) to fetch the XML server & hub host & port 						
						
XML server就相当于Tomcat服务器，顾名思义使用XML技术接收和处理客户请求，同时负责Services的 registration 和 lifecycle management (service’s creation, checking, termination etc..)						
hub server 相当于一个虚拟机？						
						
Declare, customize and launch different type of services						
Existing in File server as e.g. launcherall.mxres under ./fs/public/mxres/common/						
./launchmxj.app –l / -s / -k						
Service processes are located on the host of the launcher						
						
The features of Murex can work only when the corresponding services are running on the service host.						
						
						
						
						
						
						
						
						
						
magapp15a:/shared/home/murex$ less  .profile　　　或　　　　magapp15a:/shared/home/posop$ less  .profile						
	# Control-M related alias					
	alias jobdefs='cd /shared/opt/SCB/pos_eod/live/config/jobdefs'					
	alias ctm='cd /shared/opt/SCB/pos_eod/live'					
	alias varex='cd /shared/opt/SCB/pos_varex/live'					
	alias varu='cd /shared/opt/SCB/pos_varutils/live'					
	alias dev1utils='cd /shared/opt/pos/mxg/mxg_bodev1/scb/utils'					
	alias sgutils='cd /shared/opt/pos/mxg/mxg_bodev1/scb/utils'					
	alias dev2utils='cd /shared/opt/pos/mxg/mxg_bodev2/scb/utils'					
	alias dailyutils='cd /shared/opt/pos/mxg/mxg_bau_daily/scb/utils'					
	alias dev1rep='cd /shared/opt/pos/mxg/mxg_bodev1/scb/reports'					
	alias dev2rep='cd /shared/opt/pos/mxg/mxg_bodev2/scb/reports'					
	alias dailyrep='cd /shared/opt/pos/mxg/mxg_bau_daily/scb/reports'					
	alias l='ls -lrt'					
						
"
"						
magapp15a:/shared/home/murex$ less .env_alias						
	alias mxghome='cd /shared/opt/pos/mxg'					
						
	# Environment and POS Market Rates aliases ( 2006 )					
	# -- Murex related aliases (2006)					
	alias cdd='cd fs/public/mxres/common/dbconfig'					
	alias cdl='cd fs/public/mxres/common'					
	alias cds='cd fs/public/mxres/sites'					
						
						
						
						
						
						
						
						
						
						
						
						
						
						
http://uklpadinf01a.uk.standardchartered.com/ganglia/?c=MUREX%20G2000&m=load_one&r=hour&s=by%20name&hc=4&mc=2						
						
						
						
						
						
						
						
						
	Restart Server: must use murex not posop					
						
						
	magapp14a:/shared/opt/pos/mxg/mxg_com_daily$ ../mxg_stop_services.sh					
						
	magapp14a:/shared/opt/pos/mxg/mxg_eod$ ../mxg_start_services.sh Y YY Y Y &					
						
						
	launchmxj.app -s					
						
						
						
	MXG_FDR :-					
	Login: magapp15a: as murex					
	cd /shared/mrxdev_pos/mxg_fdr/live					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr stop					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr start					
						
						
						
						
						
						
						
mx						
	/shared/opt/SCB/dev_launchers/binaries/live/mx					


## MUREX CLIENT
On Windows Operation System Murex Client is a .bat file as shown below		
########################################		
@ECHO OFF		
		
REM Mx G2000 Client Launcher		
REM Mofify this script to match your java and server environnement		
REM For 2.2.8 and 2.2.9		
REM V2.3		
		
setlocal		
		
cd %TEMP%		
		
echo %TEMP%		
		
if exist mxjboot.jar goto create		
		
echo open gmsitnfs.uk.standardchartered.com>>getboot.cmd		
echo posop>>getboot.cmd		
echo posop123>>getboot.cmd		
echo bin>>getboot.cmd		
echo cd /shared/opt/SCB/pos_murex/live/utils/jar>>getboot.cmd		
echo cd /shared/home/murex/MxSCBTools/tomcat/webapps/mxmlmonit/clients>>getboot.cmd		
echo get mxjboot.jar>>getboot.cmd		
echo quit>>getboot.cmd		
ftp -s:getboot.cmd		
del getboot.cmd		
		
:create		
		
IF EXIST "C:\Progra~1\MXG2000\J2RE1.4.2_08" (		
SET JAVAHOME="C:\Progra~1\MXG2000\J2RE1.4.2_08"		
) ELSE (		
SET JAVAHOME="C:\Progra~1\Java\j2re1.4.2_08"		
)		
		
SET MXJ_FILESERVER_HOST=magapp15a.uk.standardchartered.com		
SET MXJ_FILESERVER_PORT=21201		
SET MXJ_SITE_NAME=default		
SET MXJ_DESTINATION_SITE_NAME=mxg2k_fdr_live		
SET MXJ_PLATFORM_NAME=MX		
SET MXJ_PROCESS_NICK_NAME=MXG2K_FDR_LIVE		
		
		
SET PATH=%JAVAHOME%\jre\bin;%JAVAHOME%\jre\bin\classic;%JAVAHOME%\bin;%JAVAHOME%\bin\classic;%PATH%		
		
SET PATH=%PATH%;bin		
SET MXJ_JAR_FILELIST=murex.download.guiclient.download		
SET MXJ_POLICY=java.policy		
SET MXJ_BOOT=mxjboot.jar		
SET MXJ_CONFIG_FILE=client.xml		
		
IF EXIST jar\%MXJ_BOOT% copy jar\%MXJ_BOOT% . >NUL		
		
title %~n0 FS:%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST%  Xml:%MXJ_SITE_NAME% /PLATF:%MXJ_PLATFORM_NAME% /NNAME:%MXJ_PROCESS_NICK_NAME%		
		
java -Xmx512m -cp %MXJ_BOOT% -Djava.security.policy=%MXJ_POLICY% -Djava.rmi.server.codebase=http://%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST% murex.rmi.loader.RmiLoader /MXJ_SITE_NAME:%MXJ_SITE_NAME% /MXJ_DESTINATION_SITE_NAME:%MXJ_DESTINATION_SITE_NAME% /MXJ_CLASS_NAME:murex.gui.xml.XmlGuiClientBoot /MXJ_PLATFORM_NAME:%MXJ_PLATFORM_NAME% /MXJ_PROCESS_NICK_NAME:%MXJ_PROCESS_NICK_NAME% /MXJ_CONFIG_FILE:%MXJ_CONFIG_FILE% %1 %2 %3 %4 %5 %6		
		
title Command Prompt		
endlocal		
pause		
########################################		
解析：该批处理程序是通过java命令行启动客户端的murex.rmi.loader.RmiLoader 程序		
		
Java命令行基本结构：		
java [ options ] class [ argument ... ] 或 java [ options ] -jar file.jar [ argument ... ]		
		
说明：		
（1）options ：命令行选项。启动器有一组标准选项，当前的运行时环境支持这些选项并且将来的版本也将支持它们。还有一组其它的非标准选项是特定于目前的虚拟机实现的，将来可能要有变化。非标准选项以 -X 打头。		
     #标准选项：		
     .-cp 类路径,指定一个用于查找类文件的列表，它由目录、 JAR 归档文件和 ZIP 归档文件组成。类路径项用分号 (;) 分隔。指定 -classpath 或 -cp 将覆盖 CLASSPATH		
     .-D属性=值,设置系统属性的值。		
     .-jar JAR归档文件的名称，注意：JAR 归档文件的名称不是启动类的类名。启动类由 Main-Class 清单头指定。JAR 文件是所有用户类的源，其它的用户类路径设置将被忽略。		
     .-verbosee或-verbose:class,显示每个所加载的类的信息。		
     .-verbose:gc ,报告每个垃圾收集事件。 		
     .-verbose:jni ,报告有关本地方法的使用和其它 Java 平台相关代码接口活动的信息。		
     .-version ,显示版本信息并退出。 		
     .-? 或-help ,显示用法信息并退出。 		
     .-X ,显示非标准选项的有关信息并退出。 		
     #非标准选项：		
     .-Xbootclasspath:自举类路径 ,指定以分号分隔的目录、 JAR 归档文件和 ZIP 归档文件列表，用以查找自举类文件。这些自举类文件用来取代 JDK 1.2 软件中所包括的自举类文件。 		
     .-Xdebug ,启动激活的调试器。Java 解释器将输出一密码供 jdb 使用。有关详细资料及程序示例，请参阅 jdb 说明。 		
     .-Xnoclassgc ,禁用类垃圾收集 		
     .-Xms$V ,指定内存分配池的初始容量$V。该值$V必须大于 1000。要使该值扩大 1000 倍，须附加上字母 k，要使该值扩大一百万倍，须附加上字母 m。缺省值为 1m。 		
     .-Xmx$v ,指定内存分配池的最大容量$V。该值$V必须大于 1000。要将它扩大 1000 倍，须附加上字母 k，要将该值扩大一百万倍，须附加上字母 m。缺省值为 16m。 		
     .-Xrunhprof[:help][:<子选项>=<值>,...] ,启用 cpu 、堆或监视器监控操作。该选项后面一般跟着一个列表，该列表由以逗号分隔的 "<子选项>=<值>" 对所组成。运行命令 java -Xrunhprof:help 可获得子选项及其缺省值的列表。 		
     .-Xrs ,减少操作系统信号的使用。 		
     .-Xcheck:jni ,对 Java 平台相关代码接口函数进行额外检查。 		
		
（2）class ：要调用的类名。 或 file.jar ：要调用的 jar 文件名。只与 -jar 一起使用。		
     缺省情况下，第一个非选项参数是要调用的类名。应当使用全限定类名。如果指定了 -jar 选项，那么第一个非选项参数是 JAR 归档文件的名称，该归档文件包含应用程序的类和资源文件以及 Main-Class 清单头指定的启动类。 		
（3）argument ：传给 main 函数的参数。 类名或 JAR 文件名后的非选项参数被传递给 main 函数。		
（4）%MXJ_PLATFORM_NAME%: 批处理中定义的变量		
%1 %2 %3 %4 %5 %6：表示参数，参数是指在运行批处理文件时在文件名后加的以空格（或者Tab）分隔的字符串。变量可以从%0到%9，%0表示批处理命令本身，其它参数字符串用%1到%9顺序表示。%~dp0：表示批处理所在目录。		
例1：C:根目录下有一批处理文件名为f.bat，内容为：@echo offformat %1如果执行C:\>f a:那么在执行f.bat时，%1就表示a:，这样format %1就相当于format a:，于是上面的命令运行时实际执行的是format a:		
例2：C:根目录下一批处理文件名为t.bat，内容为:@echo offtype %1type %2那么运行C:\>t a.txt b.txt%1 : 表示a.txt%2 : 表示b.txt于是上面的命令将顺序地显示a.txt和b.txt文件的内容。		
		
		
将必要的参数（脚本中标红的部分）通过java命令传入启动程序，不同的参数值启动了不同的murex环境的客户端程序，参数值配置规则存放在渣打的文件服务器中。		
		
Take mxg_frd environment as example		
		login file server of SCB: magapp15a.uk
		magapp15a:mxghome
		magapp15a:cd /shared/opt/pos/mxg_fdr/live
		magapp15a:/shared/opt/pos/mxg_fdr/live$ less mxg2000_settings.sh (找到参数对应的值)
		###################################
		#!/bin/sh
		
		# Murex: Jun  2003
		# launchmxj.app 2.10 version
		# Mx G2000 Environment variables setup
		# set -x
		
		TODAY="`date +'%Y%m%d_%H%M%S'`"
		
		OS_TYPE=`uname`
		OS_PLATFORM=`uname -p`
		
		if [ "$OS_TYPE" = "SunOS" ]; then
		        case ${OS_PLATFORM} in
		        sparc)
		                JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		                ;;
		        i386)
		                JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_x86
		                ;;
		        esac
		fi
		if [ "$OS_TYPE" = "AIX" ]; then
		        JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		fi
		if [ "$OS_TYPE" = "HP-UX" ]; then
		        JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		fi
		if [ "$OS_TYPE" = "Linux" ]; then
		        JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		fi
		
		# Sybase home used to locate interface files and Open Client dynamic libraries
		case ${OS_PLATFORM} in
		sparc)
		        SYBASE=/shared/opt/sybase/openclient_pos/12.5
		        ;;
		i386)
		        SYBASE=/shared/opt/sybase/openclient_pos/12.5.x86
		        ;;
		esac
		
		export SYBASE
		LD_LIBRARY_PATH=$SYBASE/OCS-12_5/lib:/usr/openwin/lib:/usr/ccs/lib:/shared/opt/S
		CB/mx_api/live/lib
		export LD_LIBRARY_PATH
		
		if [ -d "/usr/sfw/lib" ]
		then
		        if [ `echo ${LD_LIBRARY_PATH}|grep "/usr/sfw/lib"|wc -l` = 0 ]
		        then
		                LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/sfw/lib
		        fi
		fi
		
		case ${OS_PLATFORM} in
		i386)
		        LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/shared/opt/SCB/SolarisStudioX86/live
		/lib
		        ;;
		esac
		
		export LD_LIBRARY_PATH
		
		# Oracle Home used to locate Oracle Client dynamic libraries
		ORACLE_HOME=
		#ORACLE_HOME=/local/oracle/app/oracle/product/9i
		export ORACLE_HOME
		NLS_LANG=AMERICAN_AMERICA.AL32UTF8
		export NLS_LANG
		
		#MXG2000_HOME=/apps/murfx2000_new
		#export MXG2000_HOME
		
		#MXJ_JDK_OR_JRE=jdk
		#MXJ_JDK_OR_JRE=jre
		
		# Define the encrypted password for the monitor
		MXJ_PASSWORD=001000f00010003000a000d000c0
		
		# Define your default Mx G2000 File Server environment
		# Warning : Take care of the MXJ_FILESERVER_HOST in case of running on different
		 host.
		MXJ_FILESERVER_HOST=magapp15a
		MXJ_FILESERVER_PORT=10251
		
		MXJ_FILESERVER_TIME_ADJUSTMENT=1
		# Optional arguments passed to the FileServer.
		# " " are mandatory if several args.
		FILESERVER_ARGS=
		
		# Define your default Mx G2000 XmlServer environment
		#Backward compatibility with Version 2.2.9
		#Leave it blank with 2.2.10.
		MXJ_HOST=
		MXJ_PORT=
		
		# Optional arguments passed to the XmlServer such as forcing attachment to
		# a specific IP adrress. " " are mandatory if several args.
		
		XML_SERVER_ARGS="-d64 -verbose:gc -Xloggc:logs/xmls.gc.log.${TODAY} -XX:+PrintGC
		Details -XX:+PrintGCTimeStamps -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi
		.dgc.server.gcInterval=3600000"
		
		# Define your default Mx G2000 Site and Hub environment
		#Define your site name, must be defined in site.mxres.
		#Leave it by default.
		MXJ_SITE_NAME=mxg_bodev2
		
		#Define your hub name, must be defined in site.mxres.
		#Leave it by default.
		MXJ_HUB_NAME=hub_gdc
		
		# Optional arguments passed to the Hub home such as forcing attachment to
		# a specific IP adrress. " " are mandatory if several args.
		#HUB_HOME_ARGS="-Djava.rmi.server.hostname=X.X.X.X"
		HUB_HOME_ARGS="-Xms512m -Xmx1024m"
		
		# Define your default Mx G2000 MxMlExchange environment
		# Optional arguments passed to the MxMlExchange Server.
		# " " are mandatory if several args.
		# --- Added on 25 Sep 2008 to capture MXML thread process
		## MXML_SERVER_ARGS="-Xloggc:mxml_gc_log.txt -XX:+PrintGCDetails"
		
		## CR: CMKL00000183191 :
		#MXML_SERVER_ARGS="-Xloggc:mxml.gc.log.${TODAY}.$$ -XX:+PrintGCDetails"
		MXML_SERVER_ARGS=
		
		# Define your default Mx G2000 Launcher environment
		# Optional arguments passed to launcher.
		# " " are mandatory if several args.
		LAUNCHER_ARGS=
		
		# Define your default Mx G2000 Murexnet environment
		# Warning : Must be the same as specified into the murexnet.mxres configuration
		file
		# by /IPHOST:namedhost:8000
		# The Murexnet usually run on 8000 port, but you can use another one.
		MUREXNET_PORT=21203
		
		#Optional arguments passed to the murexnet such as forcing attachment to
		# a specific IP adrress or logs." " are mandatory if several args.
		MUREXNET_ARGS="/IPALTADDR:magapp15a /IPLOG"
		
		#Rticachesession Display
		RTICACHESESSION_XWIN_DISP=`echo $DISPLAY | cut -d: -f1`
		
		EXTRA_ARGS="/MXJ_PING_TIME:60000 /MXJ_PING_CHECK:600000 /MXJ_MX_PING_TIME:60000
		/MXJ_MX_PING_CHECK:600000 /MXJ_LAUNCHER_PING_TIME:60000 /MXJ_LAUNCHER_PING_CHECK
		:600000"
		##########################
		
		
%TEMP%和%MXJ_BOOT%等变量是在哪里设置的？		

## Control-M 
ControlM(ctm)在file server上的根目录：/shared/opt/SCB/pos_eod/live, cd 到用户根目录下后，ctm命令可进入该目录。								
修改JOB Draft，即JOB的定义文件，然后将其导入到ControlM中。	主要修改以下6处							
	1.   APPLICATION="1512113"  ， 改成自己的bankID							
	2.   JOBNAME="DMART_RUBICON_TRADE" ,  指向CF文件的名称 ，但JOBNAME的值要大写，							
	3.    NODEID="magapp15a"  ,  指向UNIX Server的name，　为controlM系统提供链接哪个Unix服务器							
	4.    DATACENTER="MUREXUAT" ,  指定我们的ControlM JOBS 运行在哪个server上。这个server仅在controlＭ的group级别指定的，通常我们指定它为MUREXUAT							
	5.    CMDLINE="%%BASEPATH/scripts/eod_task.pl -jobcode=%%JOBNAME   -config=env802(com_daily/vareod,magapp15a:/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/config$ ls eod_main.*.cf)  -psserver X86(delete it?)"  ,  CMDLINE中指定JOB的运行程序脚本，该脚本将读取cf文件，job脚本可以运行在不同的Murex环境中，不同的Murex环境的环境变量是不同的，-config参数指定为环境变量文件的别名（别名为文件名的一部分）。不同的环境的环境变量文件定义在fileServer: /shared/opt/SCB/pos_eod/live/config/下。							
	6.    RUN_AS="posop" , 指定以哪个UNIX用户身份，即权限去执行该ＪＯＢ的CMDLINE.							
	7.     <JOB JOBNAME="PRO_TRADE_COM" …>  ，每个JOB都有一个JOBNAME标签，其值与该JOB的cf文件中定义的JOBNAME要一致，与cf文件名也一致，但cf文件名需要小写，而JOBNAME的值要大写。							
		cd命令进入用户根目录下后，输入jobdefs就可进入/shared/opt/SCB/pos_eod/live/config/jobdefs，ControlM  JOB的cf定义文件就在这个目录下，						
	8.    <RULE_BASED_CALENDARS NAME="BATCH_DAYS" /> 标签指明该JOB根据哪个Calenda Rule去执行。							
	9.   <JOB CONFCAL="BATCH_DAYS", ….>							
								
分组，　Entity folder								
								
								
								
								
								
Login ControlM system -> Planning -> create a new workspace -> import JOB draft file								
Unload all irrelevant job in the workspace -> click Check In -> click Order       Check In 和 Order的区别？？？？？？？？？？？？？？？？？？？？？								
Monitoring -> Recent ViewPoints -> All Jobs -> Application->Hirarchy/Application中输入查询的值（预定义的值是bankID)then open-> 								
	free all your jobs including the jobs' folder							
	Tools->QUANTITATIVE Resource -> delete the current resource and then add your own new resource ,source name equals with the  <QUANTITATIVE NAME="TONYPC$"…> defined in Job draft. Cpu choos 40 in day, 							
	find the most high level job -> right click waiting info-> Apply All -> right click on the job to  run now							
								
								
								
								
只有check　job 红了可以set OK,也可以根据check job的log去fix the check job								
								
								
								
								
								
								
								
								
								
								
								
control M cf file can configure in the following files.								
								
magapp15a:/shared/opt/SCB/pos_eod/live/config$ ls *dmart*dev1*								
								
								
								
								
对于pre-product env  （如frd, var等）的control M script file is under magapp15a:/shared/opt/SCB/pos_eod/live/config/jobdefs_preprod  not magapp15a:/shared/opt/SCB/pos_eod/live/config/jobdefs								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
ENV variable 								
$POS_MUREX_BASE  =>  /shared/opt/SCB/pos_murex/live								
								
								
								
								
cf file => DB configure file								
magapp15a:/shared/opt/SCB/poscommon/live/config$ more pos_db.cf								
								
magapp15a:/shared/home/posop$ $POSCOMMON_BASE/bin/posisql PROFILE_MXG_IRDFX_DEV9								
Msg 911, Level 11, State 2:								
Server 'mx9a_sql', Line 1:								
Attempt to locate entry in sysdatabases for database 'MXG_IRDFX_DEV9' by name								
failed - no entry found under that name. Make sure that name is entered								
properly.								
1>								
q								
Solution:								
1. cd /shared/opt/SCB/dbgen/live/bin								
2. rm jerry.sed								
3. vi jerry.dbo								
4. DBUpdateCF jerry.dbo jerry.cf à jerry.sed								
5. need copy 1								
								
								
								
7. cd /shared/opt/SCB/poscommon/live/config								
								
								
								
								
								
								
								
								
								
								
                1.  open your control M draft								
                2.  remove the key word FOLDER_ORDER_METHOD="SYSTEM"								
                3.  if need open a session to align our understanding, please let me know.								
								
								
								
								
$POS_VARUTILS_BASE								
/shared/opt/SCB/pos_varutils/live								
								
								
								
								
								
								
POST WEB 								
https://posweb.gdc.standardchartered.com:8002 								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
Tomcat								
/shared/opt/tomcat/spirit-tomcat/webapps/spirit								
								
Sybase								
/shared/opt/sybase/openclient_pos/12.0.0.3								
								
/opt/bin								
/shared/home/posop/scripts								
magapp15a:/opt$  cd $POSTOOLS								
								
								
								
								
								
								
								
Set a resource name(i.e. TONYZHU$) then distribute max resource to it(i.e. 80),then modify you control M draft  (QUANTITATIVE NAME="TONYZHU$")								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
Calendar: 								
Open Control-m  then navigate Planning ->  Tolls -> Calendars -> create new/ find out existing 'CalendarName'								
								
								
								
								
								
	Sno	DB Name	Profile Name	Login	Sybase server	Port	Physical Host	Virtual Host
	1	MXG	PROFILE_MXG	MUREXDB	mxg_sql	7000	mxgdb.gdc	db046a
	2	MXML	PROFILE_MXML	MUREXDB	mxg_sql	7000	mxgdb.gdc	db046a
	3	MXG_VAR	PROFILE_VAR	MUREXDB	pos_var_sql	8100	mxgvardb.gdc	db046b
	4	MXG_RPT	PROFILE_RPT	MUREXDB	mxg_rpt_sql	7300	mxgrptdb.gdc	db050a
	5	MXG_FDR	PROFILE_FDR	MUREXDB	mxg_fdr_sql	7200	mxgfdrdb.gdc	db050b
	6	INTDB	PROFILE_INTDB	posop	pos_sql	6100	posdb.gdc	db011b
	7	MXGDM	PROFILE_MXGDM	DMDBO	mxg_rpt_sql	7300	mxgrptdb.gdc	db050a
	8	MXGDM_RPT	PROFILE_MXGDM_RPT	DMDBO	mxg_rpt_sql	7200	mxgrptdb.gdc	db050a
	9	MXGDM_FDR	PROFILE_MXGDM_FDR	DMDBO	mxg_fdr_sql	7200	mxgfdrdb.gdc	db050b
								
								
								
								
								
magapp15a:/shared/opt/pos/mxg/mxg8_1/scb/reports/dates$ ls -rlt  date_td.txt								
-rwxr--r--   1 murex    murex         40 Jan 10  2016 date_td.txt								
								
								
								
								
								
								
								
								
Where to get the latest ControlM draft?								
/dev_vob/packages/poseidon/eod_auto/controlm/mxg2k_production.xml								
[1477277@dl1101 controlm]$ ls -rlt mxg2k_production.xml								
-r--r----- 1 1377478 gmdev 6856910 Oct 13 10:03 mxg2k_production.xml								
[1477277@dl1101 controlm]$ cp mxg2k_production.xml mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ ls -rlt mxg2k_production.xml.17Oct								
-r--r----- 1 1477277 1477277 6856910 Oct 17 07:56 mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ chmod 777 mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ ls -rlt mxg2k_production.xml.17Oct								
-rwxrwxrwx 1 1477277 1477277 6856910 Oct 17 07:56 mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ scp mxg2k_production.xml.17Oct posop@magapp15a.uk:/shared/home/posop/tonyzhu/mxg2k_production.xml.17Oct								
## Murex other
想知道JOB是否还在运行		
	执行SQL	
		sp_MxLock
	config login Murex -> publisher->Datamart->Job	
		
		
		
		
Joy's murex objects export from murex		
/shared/home/murex/su_roy_yu/project/rubicon/cm0000000679433/gl2.2.2/objects		
 
## 重启launcher
	MXG_FDR :-					
	Login: magapp15a: as murex					
	cd /shared/mrxdev_pos/mxg_fdr/live					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr stop					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr start					


## /shared/mrxdev_pos/mxg_pp_services_main.sh
#!/bin/ksh

CURR_DIR=`pwd`
ENV=${1}
STATUS=${2}
MXML=${3:-"N"}
LAUNCHERS=${4:-"Y"}

case ${ENV} in 
mxg|var|fdr|rpt|purge)
	echo "valid environment provided ... continuing"
	;;
*)
	basename `dirname ${CURR_DIR} | sed -e "s;/live;;g"` | nawk -F "_" '{ print $NF }' | read -r ENV
	;;
esac

BINARY_PATH=${BINARY_PATH:-"/shared/mrxdev_pos"}

echo "-------------------------------------- [ ${ENV} ]"

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #
# function main_start()
# - - - - - - - - - - - - 
main_start() {
	./launchmxj.app -fs
	sleep 4
	case ${ENV} in
	mxg|purge)
		./launchmxj.app -xmlsnohub
		sleep 4
		./launchmxj.app -hub /MXJ_HUB_NAME:hub_gdc
		;;
	var|fdr|rpt)
		./launchmxj.app -xmls
		sleep 4
		;;
	esac

	./launchmxj.app -l
	./launchmxj.app -mxnet

	./launchmxj.app -s
}
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #
# function main_stop() {
# - - - - - - - - - - - - 
main_stop() {
	./launchmxj.app -mxnet -k
	./launchmxj.app -l -k
	case ${ENV} in 
	mxg)	
		./launchmxj.app -hub /MXJ_HUB_NAME:hub_gdc -k
		./launchmxj.app -xmlsnohub -k
		;;
	var|fdr|rpt)
		./launchmxj.app -xmls -k
		;;
	esac

	./launchmxj.app -fs -k
	./launchmxj.app -killall
	for all_hosts in `(echo magapp14a magapp15a; cat /shared/opt/SCB/dev_launchers/config/X86_ALL_HOSTS_PP.cf)`
	do
		ssh -q murex@${all_hosts} "cd ${CURR_DIR};./launchmxj.app -killall"
	done

	./launchmxj.app -s
}
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #
case ${STATUS} in
start)
	## Start main mandatory services
	main_start
	case ${ENV} in
	mxg)
		## Start MXML services for mxg env
		${BINARY_PATH}/mxg_pp_services_mxml.sh mxg start
		;;
	var|fdr|rpt|purge)
		echo "MXML services are not required for VAR/FDR/RPT environment .. continuing"
		;;
	esac
	if [ ${LAUNCHERS} == "Y" ]
	then
		## Start Launcher services for GUI and Batch
		${BINARY_PATH}/main_launchers.sh ${ENV} start

		case ${ENV} in
		fdr|rpt)
			## Start DealScanner Launcher services for FDR / RPT
			${BINARY_PATH}/dscan_launchers.sh ${ENV} start
			;;
		esac
	fi
;;
stop)
	case ${ENV} in
	fdr|rpt)
		## Stop DealScanner Launcher services for FDR / RPT
		${BINARY_PATH}/dscan_launchers.sh ${ENV} stop
		;;
	esac
	## Stop Launcher services for GUI and Batch
	${BINARY_PATH}/main_launchers.sh ${ENV} stop

	case ${ENV} in
	mxg)
		## Stop MXML services for mxg env
		${BINARY_PATH}/mxg_pp_services_mxml.sh mxg stop
		;;
	var|fdr|rpt|purge)
		echo "MXML services are not running for VAR/FDR/RPT environment .. continuing"
		;;
	esac

	## Stop main mandatory services
	main_stop

;;
test)
	./launchmxj.app -s
	;;
esac


## mxg_pp_services_mxml.sh
#!/bin/ksh

ENV=${1}
STATUS=${2}
CURR_DIR=`pwd`
BINARY_PATH=${BINARY_PATH:-"/shared/mrxdev_pos"}

case ${ENV} in
mxg|var|fdr|rpt|purge)
	echo "valid environment provided .. continuing.."
	;;
*)
	basename `dirname ${CURR_DIR} | sed -e "s;/live;;g"` | cut -d "_" -f2 | read -r ENV
	;;
esac

case ${STATUS} in
start)
	./launchmxj.app -fs
	sleep 4
	case ${ENV} in
	mxg)
		./launchmxj.app -mxml
		sleep 4
		
		./start_repository.sh
		sleep 4
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecmdex.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecri.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangedocdeal.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangefixings.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepayment.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepci.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf1.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf2.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangetds.mxres
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfinparser.mxres
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxdailyrepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfiniqrepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmarketdatarepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxssirepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxhurricanerepository.mxres
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcache.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcontribution.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxeventcapture.mxres
		sleep 4

		./mxg_start_soap_relay.sh
		sleep 4
		;;
	var|fdr|rpt)
		echo "MXML services are not required for VAR/FDR/RPT environment .. exiting"
		exit
		;;
	esac
	;;
stop)
	case ${ENV} in 
	mxg)	
		./mxg_stop_soap_relay.sh

		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxdailyrepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfiniqrepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmarketdatarepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxssirepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxhurricanerepository.mxres -k
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcache.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcontribution.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxeventcapture.mxres -k
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecmdex.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecri.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangedocdeal.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangefixings.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepayment.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepci.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf1.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf2.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangetds.mxres -k
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfinparser.mxres -k
		
		./stop_repository.sh

		./launchmxj.app -mxml -k
		./launchmxj.app -killall
		;;
	var|fdr|rpt)
		echo "MXML services are not required for VAR/FDR/RPT environment .. exiting"
		exit
		;;
	esac
;;
test)
	./launchmxj.app -s
	;;
esac

## main_launchers.sh
#!/bin/ksh
set -x

ENV=${1}
STATUS=${2}

CURR_DIR=`pwd`

BINARY_PATH=${BINARY_PATH:-"/shared/mrxdev_pos"}

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #

case ${ENV} in
mxg|var|fdr|rpt|purge)
	echo "valid environment provided .. continuing"
	;;
*)
	basename `dirname ${CURR_DIR} | sed -e "s;/live;;g"` | cut -d "_" -f2 | read -r ENV
	;;
esac
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #

SPC_FILE="launcher_${ENV}_app0"
X86_FILE="launcher_${ENV}_ukspapmrx"

case ${STATUS} in 
start)
		case ${ENV} in
		mxg)
			for sname in `echo 56 69 76`
			do
				ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres" &
				ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres" &
			done
			;;
		esac
		for sname in `echo 59 66 67 68`
		do
			ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres" &
			ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres" &
		done
		
		for x86 in `echo 01 02 03 04 05 06 08 09 10 11`
		do
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}a.mxres" &
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}b.mxres" &
		done
		
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07a.mxres" &
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13a.mxres" &
		
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07b.mxres" &
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13b.mxres" &
		
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12a.mxres" &
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14a.mxres" & 
		
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12b.mxres" &
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14b.mxres" &

		for x86new in `echo 15 16 17 18`
		do
			A_FILE="${X86_FILE}${x86new}a.mxres"
			B_FILE="${X86_FILE}${x86new}b.mxres"
			d_x86new=$((x86new-3))
			d_host="ukspadmrx${d_x86new}a"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${A_FILE}" &
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${B_FILE}" &
		done
		for x86new in `echo 19a 19b 20a 20b`
		do
			case ${x86new} in
			19a)
				d_host=12a
				;;
			19b)
				d_host=13a
				;;
			20a)
				d_host=14a
				;;
			20b)
				d_host=15a
				;;
			esac
			P_FILE="${X86_FILE}${x86new}.mxres"
			d_host="ukspadmrx${d_x86new}"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${P_FILE}" &
		done
;;
stop)
		case ${ENV} in
		mxg)
			for sname in `echo 56 69 76`
			do
				ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres -k"
				ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres -k"
			done
			;;
		esac
		for sname in `echo 59 66 67 68`
		do
			ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres -k"
			ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres -k"
		done
		for x86 in `echo 01 02 03 04 05 06 08 09 10 11`
		do
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}a.mxres -k"
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}b.mxres -k"
		done
		
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07a.mxres -k"
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13a.mxres -k"
		
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07b.mxres -k"
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13b.mxres -k"
		
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12a.mxres -k"
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14a.mxres -k"
		
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12b.mxres -k"
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14b.mxres -k"

		for x86new in `echo 15 16 17 18`
		do
			A_FILE="${X86_FILE}${x86new}a.mxres"
			B_FILE="${X86_FILE}${x86new}b.mxres"
			d_x86new=$((x86new-3))
			d_host="ukspadmrx${d_x86new}a"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${A_FILE} -k"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${B_FILE} -k"
		done
		for x86new in `echo 19a 19b 20a 20b`
		do
			case ${x86new} in
			19a)
				d_host=12a
				;;
			19b)
				d_host=13a
				;;
			20a)
				d_host=14a
				;;
			20b)
				d_host=15a
				;;
			esac
			P_FILE="${X86_FILE}${x86new}.mxres"
			d_host="ukspadmrx${d_x86new}"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${P_FILE} -k"
		done
;;
esac


## magapp15a:/shared/opt/pos/mxg/mxg_bodev1$ less ./launchmxj.app
#!/bin/sh

# Murex: 13 Feb 2004
MAJOR_VERSION=2000.2.11
MINOR_VERSION=2.11
# Mx G2000 3.1.prod Startup/Stop script for servers clients and utils.
# set -x 0

#--- Operating System ---
OS_TYPE=`uname`
PLATFORM_TYPE=`uname -p`

if [ "$OS_TYPE" = "AIX" ]; then
  MACHINE_NAME=`hostname -s`
else
  MACHINE_NAME=`hostname`
fi

echo "$1"
if [ "${MACHINE_NAME}" = "gmsitnfs" ]
then
	if [ "$1" != "-s" ]
	then
		exit
	fi
fi

##################################################

##################################################
_LS=/usr/bin/ls
_LS_L="/usr/bin/ls -l"
_PS=/usr/bin/ps

case $PLATFORM_TYPE in
sparc)
    #_TEE="/shared/opt/pos/mxg/live/mxtee -t"
    _TEE="/shared/opt/SCB/dev_launchers/scripts/mxtee -t"
    ;;
*)
    _TEE="/usr/bin/tee"
    ;;
esac

#_TEE="/shared/opt/pos/mxg/live/mxtee -t"
#_TEE=/usr/bin/tee
_AWK=/usr/bin/awk
_ECHO=echo

if [ "$OS_TYPE" = "Linux" ]; then
_LS=/bin/ls
_LS_L="/bin/ls -l"
_PS=/bin/ps
_AWK=/bin/awk
_ECHO="echo -e"
fi

_ID=`id`
if [ "$OS_TYPE" = "AIX" ]; then
  USER_NAME=`echo $_ID| sed s/\(/" "/| sed s/\)/" "/| $_AWK '{printf "%s", substr($2,1,8)}'`
else
  USER_NAME=`echo $_ID| sed s/\(/" "/| sed s/\)/" "/| $_AWK '{print $2}'`
fi

PATH=.:$PATH
export PATH

##########################################
# Common application argument environnemnt 
# Set it up here: Eventually modify the following lines according to your needs.
##########################################

# Java Debug Log Level
# 0|1|2|3|4
MXJ_LOG_LEVEL=2

# Path to store logs and PID files
LOG_PATH=logs

#Append log file at start/stop command
APPEND_LOG=1
#Create new empty log file at start/stop command
#APPEND_LOG=0

#Time loop value for the status option
#in seconds
LOOP_TIME=60

#Set File Desc default value
FD_LIMIT=2048
#FD_LIMIT=1024

# Settings for the Mx G2000 File Server.
########################################
FILESERVER_PATH=fs
MXJ_HTTP_JAR=mxjhttp.jar
# This file contains the complete path to the .jar files that are provided by the Mx G2000 File server.
# It is also used as a parameter for the other servers and the client.
MXJ_JAR_FILE=murex.download.service.download
MXJ_GUI_JAR_FILE=murex.download.guiclient.download
MXJ_FILESERVER_CONFIG_FILE=fileserver.xml

# Settings for the Mx G2000 Xml Server.
########################################
# In case of no optional flags from the setting file
# set the var to null. Do not modify here.
XML_SERVER_ARGS=

# The xmlserver.mxres allow you to specify ports to use for the xmlserver.
#MXJ_XMLSERVER_CONFIG_FILE=public.mxres.xmlserver.xmlserver.mxres

# XmlServer Stat
#MXJ_STAT_FILE_NAME=stat
#MXJ_STAT_PERIOD=10000

# Settings for the MXML Server.
########################################
# The following setting is overrwitten by the one defined in mxg2000_settings, if exists.
DEFAULT_MXML_SERVER_ARGS=""
## DEFAULT_MXML_JVM_ARGS="-Xms256M -Xmx512M -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000"
# CR: CMKL00000183191
DEFAULT_MXML_JVM_ARGS="-d64 -Xmx2048M -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000"
if [ "$OS_TYPE" = "SunOS" ]; then
   DEFAULT_MXML_JVM_ARGS="-server $DEFAULT_MXML_JVM_ARGS"
fi
MXML_SERVER_ARGS="$DEFAULT_MXML_SERVER_ARGS"
MXML_JVM_ARGS="$DEFAULT_MXML_JVM_ARGS"

# Settings for the MDCS Server.
########################################
# The following setting is overrwitten by the one defined in mxg2000_settings, if exists.
DEFAULT_MDCS_SERVER_ARGS=""
DEFAULT_MDCS_JVM_ARGS="-Xmx2g -Dsun.rmi.dgc.client.gcInterval=36000000 -Dsun.rmi.dgc.server.gcInterval=36000000"
if [ "$OS_TYPE" = "SunOS" ]; then
   DEFAULT_MDCS_JVM_ARGS="-server $DEFAULT_MDCS_JVM_ARGS"
fi
MDCS_SERVER_ARGS="$DEFAULT_MDCS_SERVER_ARGS"
MDCS_JVM_ARGS="$DEFAULT_MDCS_JVM_ARGS"

# Settings for the MDRS Server.
########################################
# The following setting is overrwitten by the one defined in mxg2000_settings, if exists.
DEFAULT_MDRS_SERVER_ARGS=""
DEFAULT_MDRS_JVM_ARGS="-Xmx512M -Dsun.rmi.dgc.client.gcInterval=36000000 -Dsun.rmi.dgc.server.gcInterval=36000000"
MDRS_SERVER_ARGS="$DEFAULT_MDRS_SERVER_ARGS"
MDRS_JVM_ARGS="$DEFAULT_MDRS_JVM_ARGS"

# Settings for all applications.
################################
# Default setting file.
SETTINGS_FILE=mxg2000_settings.sh

# Warning the MXJ_BOOT_JAR file MUST be on the same directory as the executable.
MXJ_BOOT_JAR=mxjboot.jar

# Define your default Launcher environement
# The launcherall.mxres descibe the flags you use to launch the application itself.
MXJ_CONFIG_FILE=public.mxres.common.launcherall.mxres

# Define your site name
# The site.mxres descibe the site itself.
MXJ_SITE_NAME=site1

# Define your hub name
# The site.mxres descibe the site itself.
MXJ_HUB_NAME=hub1

# Define your default MxMlExchange environement
# The file launchermxmlexchangeall.mxres describe the flags you use to launch the application itself.
MXJ_MXMLEX_CONFIG_FILE=public.mxres.common.launchermxmlexchangeall.mxres
MXJ_MXMLEX_CONFIG_FILE_SECONDARY=public.mxres.common.launchermxmlexchangesecondary.mxres
MXJ_MXMLEX_CONFIG_FILE_SPACES=public.mxres.common.launchermxmlexchangespaces.mxres

# Define your default Mx Contribution environement
MXJ_CONTRIBUTION_CONFIG_FILE=public.mxres.common.launchermxcontribution.mxres

# Define your default MDCS environement
# The file launcherwarehouse.mxres describe the flags you use to launch the application itself.
MXJ_MDCS_CONFIG_FILE=public.mxres.common.launchermxcache.mxres

# Define your default MDRS environement
# The file launchermxmarketdatarepository.mxres describe the flags you use to launch the application itself.
MXJ_MDRS_CONFIG_FILE=public.mxres.common.launchermxmarketdatarepository.mxres

#Define your default ActivityFeeder environment
# The file launchermxactivityfeeders.mxres describe the flags you use to launch the application itself.
MXJ_ACTIVITY_FEEDER_CONFIG_FILE=public.mxres.common.launchermxactivityfeeders.mxres

# Setting for murexnet.
#######################
#Port used by the murexnet, defined here if not setted in mxg2000_settings file
#(backward compatibility)
MUREXNET_PORT=8000

# In case of no optional flags from the setting file
# set the var to null. Do not modify here.
MUREXNET_ARGS=

# Settings for clients.
#######################
# Define your default client  environement
MXJ_PLATFORM_NAME=MX
MXJ_PROCESS_NICK_NAME=MX
# Define your default Client macro XML file
MXJ_SCRIPT=key.xml

###################################
# End of user definables settings # 
###################################

######################################################################
# Sourcing Setting File and Setting env.
######################################################################
Setting_Env() {  
# Java and DATABASE SERVER settings file sourced.
# Can be overwritten by option -i:setting_file

if [ ! -f `dirname $0`/$SETTINGS_FILE ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Environnement settings file: $SETTINGS_FILE not found !"
   $_ECHO "          Or : forget, -i:setting_file."
   exit 1
fi
 
. `dirname $0`/$SETTINGS_FILE
RTISESSION_XWIN_DISP=$DISPLAY
echo $RTISESSION_XWIN_DISP
RTICACHESESSION_XWIN_DISP=$DISPLAY
echo $RTICACHESESSION_XWIN_DISP

#--- OS LIBARY environment.
###########################
# Path to OS Library.
if [ "$OS_TYPE" = "SunOS" ]; then
   LD_LIBRARY_PATH=/usr/lib/lwp:$LD_LIBRARY_PATH
   export LD_LIBRARY_PATH
fi

#--- Java environment.
######################
# Path to the Java binaries, the 'libjvm.so' dynamic library
# Set it up here: Uncomment and/or modify the following lines according to your setup.

if [ "$JAVAHOME" = "" ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Please specify the Java environment "
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ "$OS_TYPE" = "SunOS" ]; then
   if [ -d $JAVAHOME/jre ]; then
   case $PLATFORM_TYPE in
        sparc )
        JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/sparc
        ;;
        i386 )
        JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/i386
        ;;
   esac
   else
   case $PLATFORM_TYPE in
        sparc )
        JAVA_LIBRARY_PATH=$JAVAHOME/lib/sparc
        ;;
        i386 )
        JAVA_LIBRARY_PATH=$JAVAHOME/lib/i386
        ;;
   esac
   fi
   PATH=$JAVAHOME/bin:$PATH
   LD_LIBRARY_PATH=$JAVA_LIBRARY_PATH:$LD_LIBRARY_PATH
   LIB_EXT=so
   export LD_LIBRARY_PATH
   # Set Numeric format for SUN
   LC_NUMERIC=en_US
   export LC_NUMERIC
fi
if [ "$OS_TYPE" = "AIX" ]; then
   JAVA_LIBRARY_PATH=$JAVAHOME/jre/bin/classic
   PATH=$JAVAHOME/jre/sh:$JAVAHOME/sh:$PATH
   LIBPATH=$JAVA_LIBRARY_PATH:$JAVAHOME/jre/bin:$LIBPATH
   LIB_EXT=a
   export LIBPATH
   # Set Numeric format for IBM
   LANG=en_US
   export LANG
   export AIXTHREAD_SCOPE=S
   export AIXTHREAD_MUTEX_DEBUG=OFF
   export AIXTHERAD_RWLOCK_DEBUG=OFF
   export AIXTHREAD_COND_DEBUG=OFF
   if [ "$LDR_CNTRL" != "" ] ; then
      LDR_CNTRL_MAXDATA=`echo $LDR_CNTRL | grep "MAXDATA"`
	  $_ECHO "WARNING: LDR_CNTRL is set with MAXDATA value:"$LDR_CNTRL
   fi
   # Settings to force JAVA to go through ipv4 stack instead of ipv6.
   export IBM_JAVA_OPTIONS="-Djava.net.preferIPv4Stack=true  ${IBM_JAVA_OPTIONS}"  
fi
if [ "$OS_TYPE" = "HP-UX" ]; then
   JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/PA_RISC2.0/hotspot
   PATH=$JAVAHOME/bin:$PATH
   SHLIB_PATH=$JAVA_LIBRARY_PATH:$SHLIB_PATH
   LIB_EXT=sl
   export SHLIB_PATH
fi
if [ "$OS_TYPE" = "Linux" ]; then
   JAVA_VENDOR=`$JAVAHOME/jre/bin/java -version 2>&1 | grep IBM` 
   if [ $? -ne 0 ] ; then 
      #Assume we are using SUN JVM
      JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/i386/client
   else
      #Assume we are using IBM JVM
      JAVA_LIBRARY_PATH=$JAVAHOME/jre/bin/client
   fi
   LD_LIBRARY_PATH=$JAVA_LIBRARY_PATH:$LD_LIBRARY_PATH
   PATH=$JAVAHOME/bin:$PATH
   LIB_EXT=so
   LC_NUMERIC=en_US
   export LIB_EXT LC_NUMERIC LD_LIBRARY_PATH
fi

export PATH

#--- Sybase environment used only by the launcher.
##################################################

if [ "$SYBASE" != "" ] ; then
case $OS_TYPE in
        SunOS )
        SYBASE_OCS=OCS-12_5
	LD_LIBRARY_PATH=$SYBASE/$SYBASE_OCS/lib:$SYBASE/lib:.:$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
        ;;
        AIX )
        SYBASE_OCS=OCS-12_5
	LIBPATH=$SYBASE/$SYBASE_OCS/lib:.:$LIBPATH
        export LIBPATH
        ;;
        HP-UX )
        SYBASE_OCS=OCS-12_5
	SHLIB_PATH=$SYBASE/$SYBASE_OCS/lib:$SYBASE/lib:.:$SHLIB_PATH
        export SHLIB_PATH
        ;;
        Linux )
	SYBASE_OCS=OCS-12_5
	LD_LIBRARY_PATH=$SYBASE/$SYBASE_OCS/lib:$SYBASE/lib:.:$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
        ;;
        * )
        $_ECHO "Warning : Do not know how to handle this OS type $OS_TYPE."
        ;;
esac
export SYBASE_OCS
fi

#--- Oracle environment.
##################################################

if [ "$ORACLE_HOME" != "" ] ; then
case $OS_TYPE in  
	SunOS )
	LD_LIBRARY_PATH=$ORACLE_HOME/lib32:/usr/ccs/lib:.:$LD_LIBRARY_PATH
	export LD_LIBRARY_PATH
	LD_LIBRARY_PATH_64=$ORACLE_HOME/lib:$LD_LIBRARY_PATH_64
	export LD_LIBRARY_PATH_64
	PATH=$ORACLE_HOME/bin:$PATH
	export PATH
    	;;
	AIX )
	LIBPATH=$ORACLE_HOME/lib32:$ORACLE_HOME/lib:.:$LIBPATH
	export LIBPATH
        ;;
	HP-UX )
        SHLIB_PATH=$ORACLE_HOME/lib32:$ORACLE_HOME/lib:.:$SHLIB_PATH
        export SHLIB_PATH
        ;;
	Linux )
        LD_LIBRARY_PATH=$ORACLE_HOME/lib32:$ORACLE_HOME/lib:/usr/ccs/lib:.:$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
        ;;
        * )
        $_ECHO "Warning : Do not know how to handle this OS type $OS_TYPE."
        ;;
esac
fi

#--- File descriptors configuration.
################################################
ulimit -n $FD_LIMIT

FILE_DESC=`ulimit -n`

if [ $FILE_DESC -lt $FD_LIMIT ]; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Can not set file descriptors to $FD_LIMIT for current shell."
   exit 1
fi
} # End of Setting_Env

#--- Copying latest mxjboot.jar if needed.
################################################

if [ -f `dirname $0`/jar/$MXJ_BOOT_JAR ] ; then
   diff ./$MXJ_BOOT_JAR jar/$MXJ_BOOT_JAR >/dev/null
   if [ $? -ne 0 ] ; then
      cp jar/$MXJ_BOOT_JAR .
   fi
fi

# End of configuration, nothing to modify below.

######################################################################
# Mx G2000 File server
######################################################################
Fileserver() {

Define_Log_File_Name fileserver
Init_Log_File

if [ "$FILESERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$FILESERVER_ARGS"
fi

cd $FILESERVER_PATH

JAVA_CMD="java $JVM_OPTION $FILESERVER_ARGS -cp $MXJ_HTTP_JAR murex.http.fileserver.FileServerJar /MXJ_PORT:$MXJ_FILESERVER_PORT /MXJ_JAR_FILE:$MXJ_JAR_FILE /MXJ_CONFIG_FILE:$MXJ_FILESERVER_CONFIG_FILE $EXTRA_ARGS"

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>../$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a ../$LOG_PATH/$LOG_FILE.log &
cd ..
Update_Log_Pid_Files $! 0

}

######################################################################
# MXJXmlserver
######################################################################
Xmlserver() {


Define_Log_File_Name xmlserver
Init_Log_File 

if [ "$XML_SERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$XML_SERVER_ARGS"
fi
if [ "$OS_TYPE" = "SunOS" ]; then
   JVM_OPTION=" -server $JVM_OPTION"
fi

JAVA_CMD="java $JVM_OPTION $XML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.home.XmlHomeStartAll /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_HUB_NAME:$MXJ_HUB_NAME.$MXJ_SITE_NAME  $EXTRA_ARGS "

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# HubHome 
######################################################################
HubHome() {
#$_ECHO "hub home:/MXJ_HUB_NAME:$MXJ_HUB_NAME.$MXJ_SITE_NAME"
Define_Log_File_Name hubhome 
Init_Log_File

if [ "$XML_SERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$HUB_HOME_ARGS"
fi
 
JAVA_CMD="java $JVM_OPTION $HUB_HOME_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.hub.HubHome /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_HUB_NAME:$MXJ_HUB_NAME.$MXJ_SITE_NAME $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MXJXMLserver  XMLSNOHUB
######################################################################
XmlserverNoHub(){

Define_Log_File_Name xmlservernohub
Init_Log_File 

if [ "$XML_SERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$XML_SERVER_ARGS"
fi
if [ "$OS_TYPE" = "SunOS" ]; then
   JVM_OPTION=" -server $JVM_OPTION"
fi

JAVA_CMD="java  $JVM_OPTION $XML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.home.XmlHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS "

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MXJTransactionManager
######################################################################
TransactionManager() {


Define_Log_File_Name transactionmanager
Init_Log_File 

JAVA_CMD="java -server $JVM_OPTION $XML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.tm.TransactionManagerHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS"

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}


######################################################################
# Launcher 
######################################################################
Launcher(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !" 
   exit 1
fi 

   JAVA_CLASSIC=

#ldd mx

Define_Log_File_Name launcher
Init_Log_File 

if [ "$LAUNCHER_ARGS" != "" ];then
   $_ECHO "Using specific args:$LAUNCHER_ARGS"
fi

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar:./jar/xml-apis.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl "

JAVA_CMD="java $JAVA_CLASSIC $JVM_OPTION $LAUNCHER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MxMlExchange  
######################################################################
Mxmlexchange(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MxMlExchange Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=


if [ "$MXML_SERVER_ARGS" != "$DEFAULT_MXML_SERVER_ARGS" ];then
   $_ECHO "Using specific MXML_SERVER_ARGS args: $MXML_SERVER_ARGS"
else
   $_ECHO "Using default MXML_SERVER_ARGS args: $MXML_SERVER_ARGS"
fi
 
if [ "$MXML_JVM_ARGS" != "$DEFAULT_MXML_JVM_ARGS" ];then
   $_ECHO "Using specific MXML_JVM_ARGS args: $MXML_JVM_ARGS"
else
   $_ECHO "Using default MXML_JVM_ARGS args: $MXML_JVM_ARGS"
fi

Define_Log_File_Name mxmlexchange
Init_Log_File

JAVA_CMD="java -verbose:gc -Xloggc:${LOG_PATH}/${LOG_FILE}.gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps $MXML_JVM_ARGS -Xbootclasspath/p:jar/xerces.jar:jar/xml-apis.jar:jar/xalan.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl $JAVA_CLASSIC $MXML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

Define_Log_File_Name mxmlexchangesecondary
Init_Log_File

JAVA_CMD="java -verbose:gc -Xloggc:${LOG_PATH}/${LOG_FILE}.gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps $MXML_JVM_ARGS -Xbootclasspath/p:jar/xerces.jar:jar/xml-apis.jar:jar/xalan.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl $JAVA_CLASSIC $MXML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE_SECONDARY $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

Define_Log_File_Name mxmlexchangespaces
Init_Log_File

JAVA_CMD="java -verbose:gc -Xloggc:${LOG_PATH}/${LOG_FILE}.gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps $MXML_JVM_ARGS -Xbootclasspath/p:jar/xerces.jar:jar/xml-apis.jar:jar/xalan.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl $JAVA_CLASSIC $MXML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE_SPACES $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MDCS
######################################################################
MDCS_CACHE(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MDCS Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=

Define_Log_File_Name mdcs
Init_Log_File

if [ "$MDCS_SERVER_ARGS" != "$DEFAULT_MDCS_SERVER_ARGS" ];then
   $_ECHO "Using specific MDCS_SERVER_ARGS args: $MDCS_SERVER_ARGS"
else
   $_ECHO "Using default MDCS_SERVER_ARGS args: $MDCS_SERVER_ARGS"
fi

if [ "$MDCS_JVM_ARGS" != "$DEFAULT_MDCS_JVM_ARGS" ];then
   $_ECHO "Using specific MDCS_JVM_ARGS args: $MDCS_JVM_ARGS"
else
   $_ECHO "Using default MDCS_JVM_ARGS args: $MDCS_JVM_ARGS"
fi

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar "

JAVA_CMD="java $MDCS_JVM_ARGS $JVM_OPTION $JAVA_CLASSIC $MDCS_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MDCS_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a /$LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MDRS
######################################################################
MDRS_ENGINE(){

if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MDRS Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

JAVA_CLASSIC=

Define_Log_File_Name mdrs
Init_Log_File

if [ "$MDRS_SERVER_ARGS" != "$DEFAULT_MDRS_SERVER_ARGS" ];then
   $_ECHO "Using specific MDRS_SERVER_ARGS args: $MDRS_SERVER_ARGS"
else
   $_ECHO "Using default MDRS_SERVER_ARGS args: $MDRS_SERVER_ARGS"

fi
if [ "$MDRS_JVM_ARGS" != "$DEFAULT_MDRS_JVM_ARGS" ];then
   $_ECHO "Using specific MDRS_JVM_ARGS args: $MDRS_JVM_ARGS"
else
   $_ECHO "Using default MDRS_JVM_ARGS args: $MDRS_JVM_ARGS"
fi

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar "

JAVA_CMD="java $MDRS_JVM_ARGS $JVM_OPTION $JAVA_CLASSIC $MDRS_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MDRS_CONFIG_FILE $EXTRA_ARGS"

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# Olk
######################################################################
Olk(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: Olk Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

OLK_EXEC_FILE=olk_exec.sh

if [ ! -f `dirname $0`/$OLK_EXEC_FILE ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Executable olk command file: $OLK_EXEC_FILE not found !"
   exit 1
fi
if [ ! -x `dirname $0`/$OLK_EXEC_FILE ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Olk command file: $OLK_EXEC_FILE not executable, change rights !"
   exit 1
fi

Define_Log_File_Name olk
Init_Log_File 
. `dirname $0`/$OLK_EXEC_FILE $EXTRA_ARGS \
2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 1

}

######################################################################
# RtImport
######################################################################
RtImport(){
#Params : session or start or stop or rtifxg
MXJ_RTIMPORT_CONFIG_PATH=public.mxres.mxcontribution.

case "$1" in
'session')

#Used to set the display value
$_ECHO $RTISESSION_XWIN_DISP >./RTISESSION_XWIN_DISP.tmp
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportsession.mxres $EXTRA_ARGS

        ;;

'rticachesession')
#set -x
#Used to set the display value
$_ECHO $RTICACHESESSION_XWIN_DISP >./RTICACHESESSION_XWIN_DISP.tmp
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportcachesession.mxres $EXTRA_ARGS

 ;;

'rticachestart')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportcachestart.mxres $EXTRA_ARGS

        ;;

'rticachestop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportcachestop.mxres $EXTRA_ARGS

        ;;

'rtiexportresults')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportexportresults.mxres $EXTRA_ARGS

        ;;

'start')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportstart.mxres $EXTRA_ARGS

        ;;
'stop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportstop.mxres $EXTRA_ARGS
        
        ;;

'fxgsession')
#set -x
#Used to set the display value
$_ECHO $RTICACHESESSION_XWIN_DISP >./RTICACHESESSION_XWIN_DISP.tmp
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportfixingsession.mxres $EXTRA_ARGS

        ;;
'fxgstart')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportfixingstart.mxres $EXTRA_ARGS

        ;;
'fxgstop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportfixingstop.mxres $EXTRA_ARGS

        ;;
esac

}

######################################################################
# MxParam
######################################################################
MxParam(){
#Params : start or stop
MXJ_MXPARAM_CONFIG_PATH=public.mxres.mxcontribution.
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: Mxparam Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

case "$1" in
'start')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxparamstop.mxres $EXTRA_ARGS

        sleep 15

        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxpmanagementstop.mxres $EXTRA_ARGS

        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxpmanagementstart.mxres $EXTRA_ARGS


        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxparamstart.mxres $EXTRA_ARGS &

        ;;
'stop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxparamstop.mxres $EXTRA_ARGS

        sleep 15

        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxpmanagementstop.mxres $EXTRA_ARGS
        
        ;;
esac

}

######################################################################
# Murexnet 
######################################################################
Murexnet() {

Define_Log_File_Name murexnet
Init_Log_File 

MXNET_MBX_CNT="mbx$MUREXNET_PORT.cnt"
MXNET_MBX_CHK="mbx$MUREXNET_PORT.chk"
MXNET_MBX_EVT="mbx$MUREXNET_PORT.evt"
MXNET_MBX_IND="mbx$MUREXNET_PORT.ind"
MXNET_MBX_NOD="mbx$MUREXNET_PORT.nod"
MXNET_MBX_ERR="mbx$MUREXNET_PORT.err"
MXNET_MBX_IERR="mbx$MUREXNET_PORT.ierr"

if [ -f $MXNET_MBX_CNT ] ; then
  rm $MXNET_MBX_CNT
fi
if [ -f $MXNET_MBX_CHK ] ; then
  rm $MXNET_MBX_CHK
fi
if [ -f $MXNET_MBX_EVT ] ; then
  rm $MXNET_MBX_EVT
fi
if [ -f $MXNET_MBX_IND ] ; then
  rm $MXNET_MBX_IND
fi
if [ -f $MXNET_MBX_NOD ] ; then
  rm $MXNET_MBX_NOD
fi
if [ -f $MXNET_MBX_ERR ] ; then
  rm $MXNET_MBX_ERR
fi
if [ -f $MXNET_MBX_IERR ] ; then
  rm $MXNET_MBX_IERR
fi
$_ECHO "Cleanup done."

if [ "$MUREXNET_ARGS" != "" ];then
   $_ECHO "Using specific args:$MUREXNET_ARGS"
fi

MXNET_CMD="./murexnet /ipaddr:$MUREXNET_PORT /stdout:stdout /stderr:stderr $MUREXNET_ARGS $EXTRA_ARGS"

$_ECHO "Murexnet cmd:\n$MXNET_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$MXNET_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MxContribution
######################################################################
MxContribution(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MxContribution Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=

if [ "$OS_TYPE" = "SunOS" ]; then
   JVM_OPTION=" -server $JVM_OPTION"
fi

#ldd mx

Define_Log_File_Name mxcontrib
Init_Log_File

JAVA_CMD="java $JVM_OPTION -Xmx150M -Xms150M $JAVA_CLASSIC -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_CONTRIBUTION_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a ../$LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MxActivityFeeder
######################################################################
MxActivityFeeder(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MxActivityFeeder Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=

#ldd mx

Define_Log_File_Name feeder
Init_Log_File

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar:./jar/xml-apis.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl "

JAVA_CMD="java $JVM_OPTION -Xmx150M -Xms150M $JAVA_CLASSIC -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_ACTIVITY_FEEDER_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# Client
######################################################################
Client() {

MXJ_BOOT_JAR=$MXJ_BOOT_JAR
java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_GUI_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.gui.xml.XmlGuiClientBoot /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS
exit $?
}

######################################################################
# Client with macro
######################################################################
ClientMacro() {

MXJ_BOOT_JAR=$MXJ_BOOT_JAR
java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.gui.api.script.ApiScript /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME \
/MXJ_SCRIPT_READ_FROM:$MXJ_SCRIPT /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS
exit $?
}


######################################################################
# Monitor
######################################################################
Monitor() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.MonitorDisplay /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD  $EXTRA_ARGS
exit $?
}
######################################################################
# Script Monitor
######################################################################
Script_Monitor() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD /MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE $EXTRA_ARGS
exit $?
}
######################################################################
# Remote Diagnostic Tool
######################################################################
MxRdt() {

OutputDir=logs/MxRDT
export OutputDir
MxRDT_LogFile=${OutputDir}/MxRDT_log.html

if [ ! -d "$OutputDir" ]; then
        mkdir "$OutputDir"
        Result=$?
        if [ "$Result" != "0" ]; then
           echo "Error creating output directory. Exit."
           exit $Result
        fi
else
        touch $MxRDT_LogFile
        Result=$?
        if [ "$Result" != "0" ]; then
          echo "Could not write in output directory. Exit. "
          exit $Result
        fi
fi

echo '<H3> <FONT COLOR="#336699"> MxRDT Log </FONT> </H4> <pre>' > $MxRDT_LogFile
. ./mxrdt 2>&1 | tee -a $MxRDT_LogFile
Result=$?
# echo '</pre>' >> $MxRDT_LogFile
exit $Result
}
######################################################################
# XmlRequestScript
######################################################################
XmlRequestScript() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL \
/MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE $EXTRA_ARGS
exit $?
}
######################################################################
# PasswordEncryption
######################################################################
PasswordEncryption() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.cryptography.GuiPassword  $EXTRA_ARGS
exit $?
}

######################################################################
#
# Beginning of script
#
######################################################################
error(){
        $_ECHO "$0: Error: $*"
}

Define_Log_File_Name() {
# Params : ID of Service 
ID=$1
LOG_FILE=

case $ID in  
        fileserver )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_FILESERVER_PORT
        ;;
        xmlserver )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_HUB_NAME.$MXJ_SITE_NAME
        ;;
        xmlservernohub )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME
        ;;
        hubhome )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_HUB_NAME.$MXJ_SITE_NAME
        ;; 
        transactionmanager )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_PORT.$MXJ_SITE_NAME
        ;;
        launcher )
                if [ "$MXJ_INSTALLATION_CODE" = "" ] ; then
                   LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_CONFIG_FILE
                else
                   LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_CONFIG_FILE.$MXJ_INSTALLATION_CODE
                fi
        ;;
        mxmlexchange )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MXMLEX_CONFIG_FILE
        ;;
        mxmlexchangesecondary )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MXMLEX_CONFIG_FILE_SECONDARY
        ;;
        mxmlexchangespaces )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MXMLEX_CONFIG_FILE_SPACES
        ;;       
        murexnet )
                LOG_FILE=$MACHINE_NAME.$ID.$MUREXNET_PORT
        ;;
        mdcs )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MDCS_CONFIG_FILE
        ;;
        mdrs )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MDRS_CONFIG_FILE
        ;;
        olk )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        mxparam )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        rtimport )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        mxcontrib )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        feeder )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_ACTIVITY_FEEDER_CONFIG_FILE
         ;;
        * )
                $_ECHO "Warning : Do not know how to handle this service."
                $_ECHO "          No way to stop it except manually."
        ;;
esac

}

Init_Log_File() {
#Params : None

if [ ! -d $LOG_PATH ] ; then
   mkdir $LOG_PATH
fi
if [ ! -d $LOG_PATH ] ; then
   $_ECHO "Mx G2000: Error : "
   $_ECHO "          Log directory :$LOG_PATH does not exist !"
   $_ECHO "          Please create it."
   exit 1
fi
if [ ! -w $LOG_PATH ] ; then
   $_ECHO "Mx G2000: Error : "
   $_ECHO "          Log directory :$LOG_PATH does not have good rights !"
   $_ECHO "          Please add write permission."
   exit 1
fi
if [ -f $LOG_PATH/$LOG_FILE.pid ] ; then
  $_ECHO "The service may already run, please check below."
  Process_Status
#  exit 1
  if [ -f $LOG_PATH/$LOG_FILE.pid ] ; then
        $_ECHO "\nThe service already run."
        exit 1
  else
        $_ECHO "\nCleanup done \nLaunching Process" 
  fi
fi

if [ $APPEND_LOG = 1 ] ; then
        $_ECHO "---------------\n" >> $LOG_PATH/$LOG_FILE.log
else 
        $_ECHO "---------------\n" > $LOG_PATH/$LOG_FILE.log
fi
$_ECHO "Start time `date` by $USER_NAME\n" >> $LOG_PATH/$LOG_FILE.log
$_ECHO "File descriptors raised to `ulimit -n` for current cmd.\n" >> $LOG_PATH/$LOG_FILE.log
$_ECHO "Java option used : $JVM_OPTION " >>$LOG_PATH/$LOG_FILE.log
java $JVM_OPTION -version 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log
$_ECHO "" >>$LOG_PATH/$LOG_FILE.log
$_ECHO "" 
}

Update_Log_Pid_Files() {
#Param 1 : The process ID number  $! 
#Param 2 : If the process is a shell exec 1 else 0

sleep 1 #Needed to give time to the process to defunct.

PID_NB=
if [ "$OS_TYPE" = "Linux" ]; then
   _PID_NB=
   PTEE_PID_NB=
   PTEE_PID_NB=`$_PS -eaf | $_AWK ' \$2 == '$!' ' | $_AWK '{ print \$3 }'`
# DSLECOMTE-DEF0023546-REMOTE_SHELL_LINUX
   _PID_NB=`$_PS -eaf | $_AWK ' \$3 == '$PTEE_PID_NB' ' | $_AWK ' \$2 != '$!' '`
# FSLECOMTE-DEF0023546-REMOTE_SHELL_LINUX
   PID_NB=`echo $_PID_NB | $_AWK '{ print \$2 }'`
else 
   PID_NB=`$_PS -eaf | $_AWK ' \$3 == '$!' ' | $_AWK '{ print \$2 }'`
fi

if [ $2 = 1 ] ; then
   PID_NB=`$_PS -eaf | $_AWK ' \$3 == '$PID_NB' ' | $_AWK '{ print \$2 }'`
fi
if [ "$PID_NB" = "" ] ; then
   $_ECHO "\n"
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          The service did not start."
   $_ECHO "          See messages on your screen or on $LOG_PATH/$LOG_FILE.log file."
   $_ECHO "\nAt `date`\n" >> $LOG_PATH/$LOG_FILE.log
   $_ECHO "  Service failed to start (message above)\n" >> $LOG_PATH/$LOG_FILE.log
   $_ECHO "\n---------------\n" >> $LOG_PATH/$LOG_FILE.log
   if [ -f $LOG_PATH/$LOG_FILE.pid ] ; then
      $_ECHO "WARNING !!\n The service may be already launched.\n"  | $_TEE -a $LOG_PATH/$LOG_FILE.log
   fi 
else
   $_ECHO "\n***\nPID:$PID_NB\n***\n" | $_TEE -a $LOG_PATH/$LOG_FILE.log
   $_ECHO "Logging stdout and stderr to $LOG_PATH/$LOG_FILE.log\n\n" 
   $_ECHO $PID_NB > $LOG_PATH/$LOG_FILE.pid
fi
}

Stop_Service() {
# Params : ID of Service 
Define_Log_File_Name $1
if [ ! -f $LOG_PATH/$LOG_FILE.pid ] ; then
   $_ECHO "Service $1 doesn't seem's to run !"
   exit 1
fi
for file in  `$_LS $LOG_PATH/$LOG_FILE.pid`
do
        FILE_OWNER=`$_LS_L $file | $_AWK '{print $3}'`
        if [ "$FILE_OWNER" != "$USER_NAME" ]; then
           $_ECHO " Not owner of $LOG_PATH/$LOG_FILE.pid"
           $_ECHO " Service not Stopped."
        else
           KILL_PID=`cat $file`
           $_ECHO "Found process pid $KILL_PID file `basename $file` "
           kill -9 $KILL_PID >/dev/null
           if [ $? -eq 0 ] ; then 
              $_ECHO "***************" >> $LOG_PATH/$LOG_FILE.log
              $_ECHO "Service stopped at `date` by $USER_NAME" | $_TEE -a $LOG_PATH/$LOG_FILE.log
              $_ECHO "***************\n" >> $LOG_PATH/$LOG_FILE.log
              rm $file
           else 
              $_ECHO "Process ID not found"
              rm $file
           fi
        fi
done

}

Kill_All() {
if [ ! -f $LOG_PATH/*.pid ] ; then
   $_ECHO "No Service running."
   exit 0
else
   for file in `$_LS $LOG_PATH/${MACHINE_NAME}*.pid`
      do
        FILE_OWNER=`$_LS_L $file | $_AWK '{print $3}'`
        if [ "$FILE_OWNER" != "$USER_NAME" ]; then
           $_ECHO " Not owner of $LOG_PATH/$file"
           $_ECHO " Service not Stopped."
        else
           LOG_FILE=`$_LS $file | sed 's/.pid//'`
           SERVICE=`echo $file | cut -d"." -f2`
           KILL_PID=`cat $file`
           $_ECHO "Found process pid $KILL_PID file `basename $file` "
           kill -9 $KILL_PID >/dev/null
           if [ $? -eq 0 ] ; then
              $_ECHO "***************" >> $LOG_FILE.log
              $_ECHO "Service stopped at `date` by $USER_NAME" | $_TEE -a $LOG_FILE.log
              $_ECHO "***************\n" >> $LOG_FILE.log
              rm $file
           else
              $_ECHO "Process ID not found, assuming process is dead"
              rm $file
           fi
        fi
      done
fi
}

Process_Status() {
# Params : None

if [ ! -f $LOG_PATH/*.pid ] ; then
   $_ECHO "No Service running."
   exit 0
fi

$_ECHO "\nFound running service(s) :"
for files in `$_LS $LOG_PATH/*.pid`
do
        SERVICE=`echo $files | sed s/\.pid//`
        SERVICE=`basename $SERVICE`
        SERVICE_LOCATION=`echo  $SERVICE | cut -d"." -f1`
        if [  "$SERVICE_LOCATION" = "$MACHINE_NAME" ] ; then
            PID_NB=`cat $files`
            # FOUND=`$_PS -eaf | $_AWK ' \$2 == '$PID_NB' '`
            FOUND=`$_PS -fp $PID_NB | grep $PID_NB`
            if [ ! "$FOUND" = "" ] ; then
            INFOS=`echo $FOUND | $_AWK '{ if ( \$5 ~ /:/ ) {  print " UID: "\$1 " PID: "\$2  " CPUTIME: "\$7 " STIME: "\$5 } else { print " UID: "\$1 " PID: "\$2  " CPUTIME: "\$8 " STIME: "\$5" " \$6 } }'` 
                   $_ECHO " $SERVICE infos : \n\t$INFOS"
            else
                   $_ECHO " $SERVICE not running, removing PID file." 
                   rm $files
                fi
        else
            $_ECHO " $SERVICE infos : \n\tService located on $SERVICE_LOCATION\n\tRun status from $SERVICE_LOCATION"
        fi 

done

} 

help() {
Setting_Env
# Params : None
        cat <<END_OF_HELP | more

$0 usage:
Version : $MAJOR_VERSION.$MINOR_VERSION

`basename $0` [ -option ] [ Param ]* [ -k | -killall ]

Options:  -i:file               : use file as the setting.
          -fs | -filserver      : launch file server service.
          -xmls | -xmlserver    : launch xmlserver server service.
          -xmlsnh | -xmlsnohub  : launch xmlserver server service without a hub.
          -hub                  : launch hub service.
          -tm                   : launch transactionmanager server service.
          -l | -launcher        : launch launcher server service.
          -mxml | -mxmlex       : launch mxmlexchange server service.
          -mxnet | -murexnet    : launch murexnet server service.
          -olk | -import        : launch olk import server service.
          -mxp | -mxparam       : launch mxparam server service.
          -rtisession | -rtimportsession : launch a session of rtimport server service.
          -rticachesession | -rtimportcachesession : launch a session of rtimport-cache server service.
          -rticache | -rtimportcache      : launch rtimport cache server service
          -rtifxgsession | -rtimportfixingsession : launch a session of rtimport fixing server service.
          -rti | -rtimport      : launch rtimport server service
          -rtifxg | -rtimportfixing : launch rtifixing import server service.
          -rtiexportresults     : launch rtimport export results service
          -mxcontrib | -mpcs    : launch mxcontribution server service.  
          -feeder               : launch activity feeder server service. 
          -mdcs | -cache        : launch MDCS cache service.
          -mdrs                 : launch MarketData repository service
          -client | -mx         : launch mx client.
          -clientmacro | -mxmacro : launch mx client in macro mode, use 
                                    /MXJ_SCRIPT_READ_FROM:script.xml to 
                                    change default script file.
          -monit | -monitor     : launch monitor.
          -smonit | -smonitor   : launch monitor in script mode (need 
                                  extra arg /MXJ_CONFIG_FILE:script_file.xml).
          -mxrdt                : remote diagnostic tool
          -xmlreq | -xmlrequest : launch xmlRequestScript class (need 
                                  extra arg /MXJ_CONFIG_FILE:xmlRequestScript.xml).
          -p | -password        : launch password encryption.
          
          -k | -kill            : option to stop service.
          -killall              : stop all running services.
          -s | -status [-loop]  : show services status [every $LOOP_TIME sec].
 
          -j:[java option] | -jopt:[java option]  : add a JVM option, can be used
                                                    as many times as options needed.
          -h | -help            : this help.

        To stop a service use the same param as the start and add -k
          ex :  to stop $0 -fs 
                   use  $0 -fs -k  
                to stop $0 -l /MXJ_CONFIG_FILE:mylauncher.mxres
                   use  $0 -l -k /MXJ_CONFIG_FILE:mylauncher.mxres
                use $0 -s to see running services.
                use $0 -killall to stop all running services.
Param used : 
        Setting File:$SETTINGS_FILE
        /MXJ_FILESERVER_HOST:$MXJ_FILESERVER_HOST
        /MXJ_FILESERVER_PORT:$MXJ_FILESERVER_PORT
        /MXJ_JAR_FILE:$MXJ_JAR_FILE
        /MXJ_PORT:$MXJ_PORT (set for backward comatibility)
        /MXJ_HOST:$MXJ_HOST (set for backward comatibility)
        /MXJ_SITE_NAME:$MXJ_SITE_NAME
        /MXJ_HUB_NAME:$MXJ_HUB_NAME
        /MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME
        /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME
        /MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE
        /MXJ_MXMLEX_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE
        /MXJ_CONTRIBUTION_CONFIG_FILE:$MXJ_CONTRIBUTION_CONFIG_FILE
        /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL
        /MXJ_SCRIPT_READ_FROM:$MXJ_SCRIPT
        MUREXNET_PORT:$MUREXNET_PORT
Specific $SETTINGS_FILE settings:
        File Server            :$FILESERVER_ARGS
        XmlServer args         :$XML_SERVER_ARGS
        Hub args               :$HUB_HOME_ARGS
        MxMlExchange args      :$MXML_SERVER_ARGS
        Launcher args          :$LAUNCHER_ARGS
        Murexnet args          :$MUREXNET_ARGS
        Real Time host display :$RTISESSION_XWIN_DISP
		MDCS args              :$MDCS_ARGS

Environment:
        LOG_PATH:$LOG_PATH
        APPEND_LOG:$APPEND_LOG
        JAVAHOME:$JAVAHOME
        SYBASE:$SYBASE
        SYBASE_OCS:$SYBASE_OCS
        ORACLE_HOME:$ORACLE_HOME

END_OF_HELP
$_ECHO "\nFile descriptors raised to `ulimit -n` for current shell.\n"
case $OS_TYPE in
        SunOS )
        $_ECHO "\nLD_LIBRARY_PATH=$LD_LIBRARY_PATH\n"
        ;;
        AIX )
        $_ECHO "\nLIBPATH=$LIBPATH\n"
        ;;
        HP-UX )
        $_ECHO "\nSHLIB_PATH=$SHLIB_PATH\n"
        ;;
        Linux )
        $_ECHO "\nLD_LIBRARY_PATH=$LD_LIBRARY_PATH\n"
        ;;
        * )
        $_ECHO "Warning : Do not know how to handle this OS type $OS_TYPE."
        ;;
esac

$_ECHO `java -version`
if [ "$SYBASE" != "" ] ; then
   if [ ! -f $SYBASE/$SYBASE_OCS/bin/isql ] ; then
      $_ECHO "Mx G2000: isql not found, please check the SYBASE environment variable"
      $_ECHO "          in the $SETTINGS_FILE script file"
   else
      $SYBASE/$SYBASE_OCS/bin/isql -v
   fi
fi
if [ "$ORACLE_HOME" != "" ] ; then
   if [ ! -f $ORACLE_HOME/bin/sqlplus ] ; then
      $_ECHO "Mx G2000: sqlplus not found, please check the ORACLE_HOME environment variable"
      $_ECHO "          in the $SETTINGS_FILE script file"
   else
      $ORACLE_HOME/bin/sqlplus -V
   fi
fi
exit 0
}

getParams() {
while [ $# != 0 ]
        do
        ARG0=$1

        PARAM=`echo $ARG0 | cut -f1 -d":"`
        VALUE=`echo $ARG0 | cut -f2- -d":"`

        if [ "$VALUE" = "" ] ; then
           help
           exit 0
        fi
        case $PARAM in
            -help | /help | -h | /h | -env | /env)
                help
                exit 0
                ;;
            -i )
                SETTINGS_FILE=$VALUE
                if [ $# -eq 1  ] ; then
                   help
                   exit 0
                fi
                ;;
            -xmls | -xmlserver )
                XMLS=1
                ;;
            -xmlsnohub | -xmlservernohub | -xmlsnh )
                XMLSNOHUB=1
                ;;
            -hub )
                HUB=1
                ;;
            -tm )
                TM=1
                ;;
            -fs | -filserver )
                FS=1
                ;;
            -l | -launcher )
                LAUNCHER=1
                ;;
            -mxml | -mxmlex )
                MXMLEX=1
                ;;
            -mdcs | -cache )
                MDCS=1
                ;;
            -mdrs )
                MDRS=1
                ;;
            -olk | -import ) 
                OLK=1
                ;;
            -mxp | -mxparam )
                MXPARAM=1
                ;;
            -rtisession | -rtimportsession )
                RTISESSION=1
                ;;
            -rticachesession | -rtimportcachesession )
                RTICACHESESSION=1
                ;;
            -rtifxgsession | -rtimportfixingsession )
                RTIFXGSESSION=1
                ;;
            -rti | -rtimport )
                RTIMPORT=1
                ;;
            -rticache | -rtimportcache )
                RTIMPORTCACHE=1
                ;;
            -rtifxg | -rtimportfixing )
                RTIFXG=1
                ;;
            -rtiexportresults )
                RTIEXPORTRESULTS=1
                ;;                
            -mxcontrib | -mpcs )
                MXCONTRIB=1
                ;;
            -feeder )
                FEEDER=1
                ;;
            -mxnet | -murexnet )
                MUREXNET=1
                ;;
            -client | -mx )
                CLIENT=1
                ;;
            -clientmacro | -mxmacro )
                CLIENTMACRO=1
                ;;
            -monit | -monitor )
                MONIT=1
                ;;
            -smonit | -smonitor )
                MXJ_CONFIG_FILE=""
                export MXJ_CONFIG_FILE
                S_MONIT=1
                ;;
            -mxrdt)
                MXRDT=1
                ;;
            -xmlreq | -xmlrequest )
                MXJ_CONFIG_FILE=""
                export MXJ_CONFIG_FILE
                XMLREQ=1
                ;;
            -p | -password )
                PASSWORD=1
                ;;
            -stop | stop | -kill | kill | -k | k )
                STOP=1
                ;;
            -stopall | stopall | -killall | killall )
                STOPALL=1
                ;;
            -status | status | -s | s )
                STATUS=1
                ;;
            -loop )
                LOOP=1
                ;;
            -j | j | -jopt | jopt )
                JVM_OPTION=$JVM_OPTION" "$VALUE
                $_ECHO "Using JVM option:$VALUE "
                ;;
            /MXJ_PORT | -MXJ_PORT )
                MXJ_PORT=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_HOST | -MXJ_HOST )
                MXJ_HOST=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_SITE_NAME | -MXJ_SITE_NAME )
                MXJ_SITE_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_HUB_NAME | -MXJ_HUB_NAME )
                MXJ_HUB_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_FILESERVER_HOST | -MXJ_FILESERVER_HOST )
                MXJ_FILESERVER_HOST=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_FILESERVER_PORT | -MXJ_FILESERVER_PORT )
                MXJ_FILESERVER_PORT=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_JAR_FILE | -MXJ_JAR_FILE )
                MXJ_JAR_FILE=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_GUI_JAR_FILE | -MXJ_GUI_JAR_FILE )
                MXJ_GUI_JAR_FILE=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_CONFIG_FILE | -MXJ_CONFIG_FILE )
                NBP=`echo $VALUE | $_AWK -F. '{ print NF }'`
                if [ "$NBP" = "2" ] && [ "$LAUNCHER" = "1" ] ; then
                   VALUE=`echo $MXJ_CONFIG_FILE | $_AWK  -F. '{ print \$1"."\$2"."\$3"." }'`$VALUE
                   echo "Guessing config file is $VALUE"
                fi 
                if [ "$NBP" = "1" ] && [ "$LAUNCHER" = "1" ] ; then
                   VALUE=`echo $MXJ_CONFIG_FILE | $_AWK  -F. '{ print \$1"."\$2"."\$3"." }'`${VALUE}.mxres
                   echo "Guessing config file is $VALUE"
                fi
                MXJ_CONFIG_FILE=$VALUE
                MXJ_MXMLEX_CONFIG_FILE=$VALUE 
                MXJ_CONTRIBUTION_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_MXMLEX_CONFIG_FILE | -MXJ_MXMLEX_CONFIG_FILE )
                MXJ_MXMLEX_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_MDCS_CONFIG_FILE | -MXJ_MDCS_CONFIG_FILE )
                MXJ_MDCS_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_MDRS_CONFIG_FILE | -MXJ_MDRS_CONFIG_FILE )
                MXJ_MDRS_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_CONTRIBUTION_CONFIG_FILE | MXJ_CONTRIBUTION_CONFIG_FILE )
                MXJ_CONTRIBUTION_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_ACTIVITY_FEEDER_CONFIG_FILE | MXJ_ACTIVITY_FEEDER_CONFIG_FILE )
                MXJ_ACTIVITY_FEEDER_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_PLATFORM_NAME | -MXJ_PLATFORM_NAME )
                MXJ_PLATFORM_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_PROCESS_NICK_NAME | -MXJ_PROCESS_NICK_NAME )
                MXJ_PROCESS_NICK_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_LOG_LEVEL| -MXJ_LOG_LEVEL)
                MXJ_LOG_LEVEL=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_POLICY_FILE | -MXJ_POLICY_FILE )
                MXJ_POLICY_FILE=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_SCRIPT_READ_FROM| -MXJ_SCRIPT_READ_FROM)
                MXJ_SCRIPT=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_INSTALLATION_CODE| -MXJ_INSTALLATION_CODE )
                MXJ_INSTALLATION_CODE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                EXTRA_ARGS="$EXTRA_ARGS $ARG0"
                ;;
            /MUREXNET_PORT| -MUREXNET_PORT| MUREXNET_PORT)
                MUREXNET_PORT=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXRDT_OUTPUT_FILENAME| -MXRDT_OUTPUT_FILENAME)
                MXRDT_OUTPUT_FILENAME=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;

            * )
                EXTRA_ARGS="$EXTRA_ARGS $ARG0"
                ;;
        esac
        shift
done
}

#Main

XMLS=0
XMLSNOHUB=0
HUB=0
TM=0
FS=0
LAUNCHER=0
MXMLEX=0
OLK=0
MDCS=0
MDRS=0
MXPARAM=0
RTISESSION=0
RTICACHESESSION=0
RTIFXGSESSION=0
RTIMPORT=0
RTIMPORTCACHE=0
RTIFXG=0
RTIEXPORTRESULTS=0
MXCONTRIB=0
FEEDER=0
MUREXNET=0
CLIENT=0
CLIENTMACRO=0
MONIT=0
S_MONIT=0
MXRDT=0
PASSWORD=0
STOP=0
STOPALL=0
STATUS=0
LOOP=0
XMLREQ=0
JVM_OPTION="-Xmx512M"

if [ "$OS_TYPE" = "AIX" ]; then
  JVM_OPTION=$JVM_OPTION" -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl "
  JVM_OPTION=$JVM_OPTION" -Djavax.xml.parsers.DocumentBuilderFactory=org.apache.xerces.jaxp.DocumentBuilderFactoryImpl "
  JVM_OPTION=$JVM_OPTION" -Djavax.xml.transform.TransformerFactory=org.apache.xalan.processor.TransformerFactoryImpl "
fi

MXJ_INSTALLATION_CODE=""
EXTRA_ARGS=

if [ $# = 0 ] ; then
        help;
        exit 0 ;
fi

Setting_Env
getParams $*

if [ "$EXTRA_ARGS" != "" ] ; then
        $_ECHO "Extra arguments used: $EXTRA_ARGS"
fi

if [ $FS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service fileserver 
        else
                Fileserver $*;
        fi
fi
if [ $TM = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service transactionmanager
        else
                TransactionManager $*;
        fi
fi
if [ $XMLS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service xmlserver
        else
                Xmlserver $*;
        fi
fi
if [ $XMLSNOHUB = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service xmlservernohub
        else
                XmlserverNoHub $*;
        fi
fi

if [ $HUB = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service hubhome
        else
                HubHome $*;
        fi
fi

if [ $LAUNCHER = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service launcher
        else
                Launcher $*;
        fi
fi
if [ $MXMLEX = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mxmlexchange
                Stop_Service mxmlexchangesecondary
                Stop_Service mxmlexchangespaces
        else
                Mxmlexchange $*;
        fi
fi
if [ $MDCS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mdcs
        else
                MDCS_CACHE $*;
        fi
fi
if [ $MDRS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mdrs
        else
                MDRS_ENGINE $*;
        fi
fi
if [ $OLK = 1 ] ; then
        if [ $STOP = 1 ] ; then
               Stop_Service olk
        else
                Olk $*;
        fi
fi
if [ $MXPARAM = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service mxparam
                MxParam stop $*;
        else
                MxParam start $*;
        fi
fi
if [ $RTISESSION = 1 ] ; then
                RtImport session $*;
fi
if [ $RTICACHESESSION = 1 ] ; then
                RtImport rticachesession $*;
fi
if [ $RTIFXGSESSION = 1 ] ; then
                RtImport fxgsession $*;
fi

if [ $RTIMPORT = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service rtimport
                RtImport stop $*;
        else
                RtImport start $*;
        fi
fi
if [ $RTIMPORTCACHE = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service rtimportcache
                RtImport rticachestop $*;
        else
                RtImport rticachestart $*;
        fi
fi
if [ $RTIEXPORTRESULTS = 1 ] ; then
                RtImport rtiexportresults $*;
fi

if [ $RTIFXG = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service rtimport
                RtImport fxgstop $*;
        else
                RtImport fxgstart $*;
        fi
fi
if [ $MXCONTRIB = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mxcontrib
        else
                MxContribution $*;
        fi
fi
if [ $FEEDER = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service feeder
        else
                MxActivityFeeder $*;
        fi
fi
if [ $MUREXNET = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service murexnet
        else
                Murexnet $*;
        fi
fi
if [ $CLIENT = 1 ] ; then
        Client $*;
fi
if [ $CLIENTMACRO = 1 ] ; then
        ClientMacro $*;
fi
if [ $MONIT = 1 ] ; then
        Monitor $*;
fi
if [ $S_MONIT = 1 ] ; then
        Script_Monitor $*;
fi
if [ $MXRDT = 1 ] ; then
        MxRdt
fi
if [ $XMLREQ = 1 ] ; then
        XmlRequestScript $*;
fi
if [ $PASSWORD = 1 ] ; then
        PasswordEncryption $*;
fi
if [ $STATUS = 1 ] ; then

        Process_Status $*;
        while [ $LOOP = 1 ] 
        do
                $_ECHO "\nSleeping $LOOP_TIME sec"
                sleep $LOOP_TIME
                Process_Status $*
        done
fi
if [ $STOPALL = 1 ] ; then
        Kill_All
fi
#END of SCRIPT



## DB TABLES Related with  Datamart reporting 
. TAB_UDF_DEALALL_REP
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
----------------	--------------	------------------------------	---------	-------	--------	--------	---------------------------	------------	-------------------	-----------
	TIMESTAMP	timestamp	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_IDENTITY	numeric	5	9	0	FALSE	(null)	(null)	(null)	TRUE
	M_AAG_BNDRY	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_AAG_PERIOD	numeric	3	4	0	FALSE	(null)	(null)	(null)	FALSE
	M_AAG_TRNCHE	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_CONFO_ID	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_CSA_EXCLUD	char	2	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_INDEP_AMT	numeric	9	17	2	FALSE	(null)	(null)	(null)	FALSE
	M_IND_AMTCCY	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_MARKETER	char	30	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_MX_REF_JOB	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_NACK_AMEND	char	30	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_NACK_COMME	char	30	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_NACK_REASO	char	30	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_NACK_REQUE	char	30	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Trade Number	M_NB	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_NOTIONAL	numeric	10	20	5	FALSE	(null)	(null)	(null)	FALSE
	M_PRICE_CAP	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_REF_DATA	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_SRC_SYSTEM	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_STRUCT_ID	numeric	8	15	0	FALSE	(null)	(null)	(null)	FALSE
	M_CFTC_BFH	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_CNTP_SNAME	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_LCH_CLR_ID	char	30	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_OLD_CPTY	char	15	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_EXT_REF_NB	char	40	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_NOVATED	char	20	(null)	(null)	FALSE	TAB_UDF_DE_M_NOVA_493658221	(null)	(null)	FALSE
	M_NOV_DT	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_NOV_TRD_DT	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_ORIG_CPTY	char	15	(null)	(null)	FALSE	TAB_UDF_DE_M_ORIG_509658278	(null)	(null)	FALSE
	M_OLD_NB_EXT	numeric	6	10	0	FALSE	TAB_UDF_DE_M_OLD__525658335	(null)	(null)	FALSE
										
										
Comment: 										
Those 5 classified production's common attributes are collected in this table. 是5类产品UDF表中字段的交集										
///DM_DATES_REP
	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
	M_CONSO_DATE   	datetime	8			1				0
	M_DATE_REP     	datetime	8			1				0
	M_MX_REF_JOB   	numeric	6	10	0	0				0
	M_REF_DATA     	numeric	6	10	0	0				0
The new rolled date  (= trl + U ),System date	M_REP_DATE0    	datetime	8			1				0
New rolled date -2	M_REP_DATE_1   	datetime	8			1				0
New rolled date -1(previous business date), EOD 	M_REP_DATE_2   	datetime	8			1				0
	M_DATE_TPLS2   	datetime	8			1				0
										
	sp_help DM_DATES_REP									
										
Note:  Every day system need to roll date. Once roll date completed this table will be update. 										
////TABLE#LIST#CLASSIFI_REP
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
	M__ID_         	numeric	6	10	0	0				0
	M_OPERATION    	char	1			0				0
	M_STP          	char	1			0				0
	M_USER_GRP     	char	30			0				0
										
										
Murex Security Matrix 表，描述各个group是否有operation的权限										
///TAB_TRN_IRD_PL1_REP（PL1表）
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
---------------	---------------	---------	---------	-------	--------	--------	---------------	------------	-------------------	-----------
	TIMESTAMP	timestamp	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_IDENTITY	numeric	5	9	0	FALSE	(null)	(null)	(null)	TRUE
	M_C_CUR_PL	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_C_PL_DT	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_DATE_REP0	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_DATE_REP1	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
EOD date (闭市日期，往往是system date -1)	M_DATE_REP2	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_H_ACCCPN	numeric	11	24	4	FALSE	(null)	(null)	(null)	FALSE
	M_H_MPCLEAN	numeric	11	23	4	FALSE	(null)	(null)	(null)	FALSE
underlying instrument	M_INSTRUMENT	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_INTVAL2	numeric	9	19	5	FALSE	(null)	(null)	(null)	FALSE
	M_MP_R2	numeric	6	11	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_RF1	numeric	6	11	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_RF2	numeric	6	11	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_SPOT2	numeric	8	16	6	FALSE	(null)	(null)	(null)	FALSE
	M_MP_SPOT22	numeric	8	16	6	FALSE	(null)	(null)	(null)	FALSE
	M_MP_SPOTC2	numeric	8	16	11	FALSE	(null)	(null)	(null)	FALSE
	M_MP_SWAP2	numeric	6	12	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_UDDRP11	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_UDDRP12	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_UDDRP21	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_UDDRP22	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_UPRC2	numeric	10	21	11	FALSE	(null)	(null)	(null)	FALSE
	M_MP_VOL1	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_MP_VOL2	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_MRPL_ONB	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
Portfolio name	M_MX_PFOLIO	char	16	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_MX_REF_JOB	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
Trade Number	M_NB	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
External Sequential Number(Trade number in another system)	M_NB_EXT	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
Currency of Leg 1	M_PLIRDCUR1	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Currency of Leg 2	M_PLIRDCUR2	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Fixed/Variable indicator for Leg 1	M_PLIRDFIX1	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Fixed/Variable indicator for Leg 2	M_PLIRDFIX2	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_PLIRDFPV10	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_PLIRDFPV11	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_PLIRDFPV12	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_PLIRDFPV20	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_PLIRDFPV21	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_PLIRDFPV22	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CGR1	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CGR2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CGU1	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CGU2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CSFI2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CSNFCP2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_CSNFRV2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FE2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FEFI2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FMV2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FPFCP2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FPFRV2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FPNFCP2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FPNFRV2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_FTFI2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_MVFCP2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_MVFRV2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_NFMV2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_RVR2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_RVU2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PR_CTYACR2	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
Father trade number of curenttrade number	M_QTY_INDEX	numeric	3	3	0	FALSE	(null)	(null)	(null)	FALSE
	M_REF_DATA	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
Initial accrued coupon	M_TP_ACCCPN	numeric	6	12	7	FALSE	(null)	(null)	(null)	FALSE
Initial accrued coupon (2)	M_TP_ACCCPN2	numeric	6	12	7	FALSE	(null)	(null)	(null)	FALSE
A/B/D/E/Q	M_TP_AE	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
经纪人？	M_TP_BROKER0	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
经纪人？	M_TP_BROKER1	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
经纪人？	M_TP_BROKER2	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
经纪人？	M_TP_BROKER3	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
B or S	M_TP_BUY	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_BUY_E	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMDFYS0	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMDFYS1	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
First settlement date (COM futures)- moves with Maturity date	M_TP_CMGSF0	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_CMGSF1	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_CMLDIR0	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMLDIR1	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMULAB0	char	25	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMULAB1	char	25	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMUQ0	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMUQ1	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMUQU0	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CMUQU1	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Counter party	M_TP_CNTRP	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CNTRPFN	char	255	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CNTRPLB	char	50	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
C or F or  P or S 	M_TP_CP	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_CREATOR	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_TP_DELIVER	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_DIGPAY	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
trade过期日期 	M_TP_DTEEXP	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_DTEFST	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_DTEFXGF	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_DTEFXGL	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_DTEPMT	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_DTEPMT2	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
System Creation Date (when trade was created)	M_TP_DTESYS	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
trade交易日期	M_TP_DTETRN	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_DTEUND	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
C or D	M_TP_DVCS	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
城市名称i.e. SHUZHOU/NEWYORK	M_TP_ENTITY	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
i.e. AED	M_TP_FEECUR0	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
i.e. TWO	M_TP_FEECUR1	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FEECUR2	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FEECUR3	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FXBRWED	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_FXBRWSD	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_FXCTPDE	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FXCTPFF	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FXCTPNU	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FXCTSFF	numeric	8	15	7	FALSE	(null)	(null)	(null)	FALSE
N 不是internal trade; Y 是internal trade. 注：murex 内部portfolio to portfolio 的trade叫internal trade(没有conterparty).	M_TP_INT	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_IPAY	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
XG3	M_TP_IPAYCUR	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
i.e. 0.09999999	M_TP_IQTY	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_LQTY1	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_LQTY2	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_LQTYS1	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_LQTYS2	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_MKTNDX	char	40	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_MOPCRT	numeric	2	1	0	FALSE	(null)	(null)	(null)	FALSE
	M_TP_MOPCRTD	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_MOPCRTL	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_MOPCRTS	char	5	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
"   when PL.M_TP_MOPLST=1
    then 'Exercise'
    when PL.M_TP_MOPLST=2
    then 'Expiry'
    when PL.M_TP_MOPLST=3
    then 'Early Termination'
    when PL.M_TP_MOPLST=4
    then 'Netting'
    when PL.M_TP_MOPLST=5
    then 'Restructure'
    when PL.M_TP_MOPLST=6
    then 'Cancel & reissue'
    when PL.M_TP_MOPLST=7
    then 'Cancel'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =1
    then 'Exercise'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =2
    then 'Expiry'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =3
    then 'Early Termination'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =4
    then 'Netting'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =5
    then 'Restructure'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =6
    then 'Cancel & reissue'
    when PL.M_TP_MOPLST =0
    and HDR.M_MOP_CREAT =7
    then 'Cancel'
    else 'UNKNOWN'"	M_TP_MOPLST	numeric	2	1	0	FALSE	(null)	(null)	(null)	FALSE
	M_TP_MOPLSTD	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
Market operation(What has been done on the current trade)	M_TP_MOPLSTL	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
i.e. 35078	M_TP_NBLTI	numeric	6	10	0	FALSE	(null)	(null)	(null)	FALSE
	M_TP_NOMCUR	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
i.e.  1.30	M_TP_NOMINAL	numeric	9	19	2	FALSE	(null)	(null)	(null)	FALSE
	M_TP_OBAR1	numeric	8	16	6	FALSE	(null)	(null)	(null)	FALSE
	M_TP_OBAR2	numeric	8	16	6	FALSE	(null)	(null)	(null)	FALSE
	M_TP_OBARTYP	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_PAYCUR	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
port folio	M_TP_PFOLIO	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_QTYEQ	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTACR02	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTACR12	numeric	8	16	2	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTAMO	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTCCP02	numeric	12	25	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTCCP12	numeric	12	25	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTCUR0	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTCUR1	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTDXC02	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTDXC12	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTDXG02	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTDXG12	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTDXP02	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTDXP12	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTFV0	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTFV1	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTFXC02	numeric	9	19	6	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTFXC12	numeric	9	19	6	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTFXSPT	numeric	8	16	4	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTMAT0	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTMAT1	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_TP_RTMBDC0	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTMBDC1	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTMNDX0	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTMNDX1	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTPR0	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTPR1	char	1	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTRFCT0	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_TP_RTRFCT1	numeric	5	9	4	FALSE	(null)	(null)	(null)	FALSE
	M_TP_SECCOD	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_SECISSU	char	40	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_STATUS0	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_STATUS1	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Current trade statrus(LIVE or DEAD)	M_TP_STATUS2	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
现货价格	M_TP_STRIKE	numeric	9	18	8	FALSE	(null)	(null)	(null)	FALSE
	M_TP_STRIKE2	numeric	9	18	8	FALSE	(null)	(null)	(null)	FALSE
交易策略（如Breakouts，Retracements，Novation）	M_TP_STRTGY	char	15	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
Trader name	M_TP_TRADER	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_TYPO	char	20	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_UQTY	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_UQTYEQ	char	10	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_VALSTAT	char	4	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
"Production (COM  
CRD  
CURR 
IRD  
SCF)"	M_TRN_FMLY	char	5	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
classify as per production	M_TRN_GRP	char	5	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
further classify	M_TRN_TYPE	char	5	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TP_FXUND	char	3	(null)	(null)	FALSE	(null)	(null)	(null)	FALSE
	M_TRN_GTYPE	numeric	3	3	0	FALSE	(null)	(null)	(null)	FALSE
	M_MRPL_DATE	datetime	8	(null)	(null)	TRUE	(null)	(null)	(null)	FALSE
	M_PL_CSNF2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_PL_MVNFCP2	numeric	11	23	6	FALSE	(null)	(null)	(null)	FALSE
	M_TP_LQTY32	numeric	11	24	8	FALSE	(null)	(null)	(null)	FALSE
										
	 187 record(s) selected [Fetch MetaData: 1517/ms] [Fetch Data: 133/ms] 									
										
										
										
										
Comment :   TAB_ALLTRNRP_PL1_REP is too many records. So spilit this big PL1 table into 5 son PL1  table as per 5 different productions. i.e. DMDBO.TAB_TRN_SCF_PL1_REP										

///TAB_TRN_IRD_PL2_REP(PL2表)
	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
	M_DATE_REP0    	datetime	8			1				0
	M_DATE_REP1    	datetime	8			1				0
	M_DATE_REP2    	datetime	8			1				0
	M_MP_SPOT0     	numeric	8	16	6	0				0
	M_MP_SPOT1     	numeric	8	16	6	0				0
	M_MP_SPOT2     	numeric	8	16	6	0				0
	M_MRPL_ONB     	numeric	6	10	0	0				0
	M_MX_PFOLIO    	char	16			0				0
	M_MX_REF_JOB   	numeric	6	10	0	0				0
	M_NB           	numeric	6	10	0	0				0
	M_NB_EXT       	numeric	6	10	0	0				0
	M_NB_INIT      	numeric	6	10	0	0				0
	M_NB_TZSYS     	numeric	2	2	0	0				0
	M_NB_TZTRN     	numeric	2	2	0	0				0
	M_PLIRDACS12   	numeric	8	16	2	0				0
	M_PLIRDACS22   	numeric	8	16	2	0				0
	M_PLIRDFPV12   	numeric	8	16	2	0				0
	M_PLIRDFPV22   	numeric	8	16	2	0				0
	M_PLIRDNFC12   	numeric	8	16	2	0				0
	M_PLIRDNFC22   	numeric	8	16	2	0				0
	M_PLIRDNFP12   	numeric	8	16	2	0				0
	M_PLIRDNFP22   	numeric	8	16	2	0				0
	M_PL_CGRA2     	numeric	11	23	6	0				0
	M_PL_CGUA2     	numeric	11	23	6	0				0
	M_PL_INSCUR    	char	20			0				0
	M_PL_KEY1      	char	30			0				0
	M_PL_NFMVTH2   	numeric	11	23	6	0				0
	M_QTY_INDEX    	numeric	3	3	0	0				0
	M_REF_DATA     	numeric	6	10	0	0				0
	M_RSKSECTION   	char	20			0				0
	M_S_BPV        	numeric	9	19	4	0				0
	M_S_CAPF2      	numeric	9	19	5	0				0
	M_TP_ACCCPND   	numeric	6	12	7	0				0
	M_TP_ACCSCT    	char	15			0				0
	M_TP_BOSGN     	char	10			0				0
	M_TP_BUY       	char	1			0				0
	M_TP_CMCLAB    	char	41			0				0
	M_TP_CMCLST    	datetime	8			1				0
	M_TP_CMCMAT    	char	21			0				0
	M_TP_CMGQL00   	numeric	9	18	4	0				0
	M_TP_CMGQL02   	numeric	9	18	4	0				0
	M_TP_CMGQTY0   	numeric	9	18	4	0				0
	M_TP_CMIQC0    	char	3			0				0
	M_TP_CMIQU0    	char	20			0				0
	M_TP_CMLQU0    	char	20			0				0
	M_TP_CMUCLB0   	char	25			0				0
	M_TP_CMUPUB0   	char	20			0				0
	M_TP_CNTRP     	char	15			0				0
	M_TP_CUTOFF    	char	10			0				0
	M_TP_DIGPAYC   	char	3			0				0
	M_TP_DTEFLWF   	datetime	8			1				0
	M_TP_DTEFLWL   	datetime	8			1				0
	M_TP_DTELBL    	char	16			0				0
	M_TP_DTELST    	datetime	8			1				0
	M_TP_DTESYS    	datetime	8			1				0
	M_TP_DTETRN    	datetime	8			1				0
	M_TP_DTEULBL   	char	16			0				0
	M_TP_DTEUNDI   	datetime	8			1				0
	M_TP_DVCS      	char	1			0				0
	M_TP_FEE0      	numeric	8	16	3	0				0
	M_TP_FXBASE    	char	3			0				0
	M_TP_FXBRSTL   	char	1			0				0
	M_TP_FXCTQDT   	char	3			0				0
	M_TP_FXCTQGA   	char	3			0				0
	M_TP_FXHDT     	numeric	4	6	2	0				0
	M_TP_FXQTOCU   	char	3			0				0
	M_TP_FXUND     	char	3			0				0
	M_TP_INITSD    	char	1			0				0
	M_TP_IPAYDTE   	datetime	8			1				0
	M_TP_IQTY      	numeric	11	24	8	0				0
	M_TP_IQTYS     	numeric	11	24	8	0				0
	M_TP_MOPCRT    	numeric	2	1	0	0				0
	M_TP_NOMCUR    	char	3			0				0
	M_TP_NOMINAL   	numeric	9	19	2	0				0
	M_TP_OBARRB1   	numeric	8	16	6	0				0
	M_TP_OBARRB2   	numeric	8	16	6	0				0
	M_TP_PFOLIO    	char	15			0				0
	M_TP_PRICE     	numeric	12	25	8	0				0
	M_TP_PRICED    	numeric	12	25	8	0				0
	M_TP_QTYEQ     	numeric	11	24	8	0				0
	M_TP_QTYEQS    	numeric	11	24	8	0				0
	M_TP_RTAMC01   	numeric	8	16	2	0				0
	M_TP_RTAMC02   	numeric	8	16	2	0				0
	M_TP_RTCAP0    	numeric	9	19	2	0				0
	M_TP_RTCAP1    	numeric	9	19	2	0				0
	M_TP_RTCCP00   	numeric	12	25	8	0				0
	M_TP_RTCCP01   	numeric	12	25	8	0				0
	M_TP_RTCCP02   	numeric	12	25	8	0				0
	M_TP_RTCCP10   	numeric	12	25	8	0				0
	M_TP_RTCCP11   	numeric	12	25	8	0				0
	M_TP_RTCCP12   	numeric	12	25	8	0				0
	M_TP_RTDPC02   	datetime	8			1				0
	M_TP_RTDPC12   	datetime	8			1				0
	M_TP_RTDXG00   	datetime	8			1				0
	M_TP_RTDXG01   	datetime	8			1				0
	M_TP_RTDXG02   	datetime	8			1				0
	M_TP_RTDXG10   	datetime	8			1				0
	M_TP_RTDXG11   	datetime	8			1				0
	M_TP_RTDXG12   	datetime	8			1				0
	M_TP_RTFICC0   	char	3			0				0
	M_TP_RTFICC1   	char	3			0				0
	M_TP_RTFV0     	char	1			0				0
	M_TP_RTFV1     	char	1			0				0
	M_TP_RTMAT0    	datetime	8			1				0
	M_TP_RTMAT1    	datetime	8			1				0
	M_TP_RTMCLG0   	char	10			0				0
	M_TP_RTMCLG1   	char	10			0				0
	M_TP_RTMCLP0   	char	10			0				0
	M_TP_RTMCLP1   	char	10			0				0
	M_TP_RTMCNV0   	char	15			0				0
	M_TP_RTMCNV1   	char	15			0				0
	M_TP_RTMFRC0   	char	15			0				0
	M_TP_RTMFRC1   	char	15			0				0
	M_TP_RTMFRF0   	char	10			0				0
	M_TP_RTMFRF1   	char	10			0				0
	M_TP_RTMFRG0   	char	10			0				0
	M_TP_RTMFRG1   	char	10			0				0
	M_TP_RTMFRP0   	char	15			0				0
	M_TP_RTMFRP1   	char	15			0				0
	M_TP_RTMGC02   	numeric	5	9	4	0				0
	M_TP_RTMGC12   	numeric	5	9	4	0				0
	M_TP_RTMMRG0   	numeric	5	9	4	0				0
	M_TP_RTMMRG1   	numeric	5	9	4	0				0
	M_TP_RTMRTE0   	numeric	9	19	6	0				0
	M_TP_RTMRTE1   	numeric	9	19	6	0				0
	M_TP_RTVF02    	numeric	8	16	2	0				0
	M_TP_RTVF12    	numeric	8	16	2	0				0
	M_TP_RTVLC02   	numeric	9	19	6	0				0
	M_TP_RTVLC12   	numeric	9	19	6	0				0
	M_TP_RTVNF02   	numeric	8	16	2	0				0
	M_TP_RTVNF12   	numeric	8	16	2	0				0
	M_TP_RTXCHC    	char	1			0				0
	M_TP_RTXCHF    	char	1			0				0
	M_TP_RTXCHI    	char	1			0				0
	M_TP_SECCUR    	char	3			0				0
	M_TP_SECISS    	char	40			0				0
	M_TP_SECLBL    	char	15			0				0
	M_TP_SPOTN     	char	20			0				0
	M_TP_SPTFWD    	char	1			0				0
	M_TP_STATUS0   	char	10			0				0
	M_TP_STATUS1   	char	10			0				0
	M_TP_STATUS2   	char	10			0				0
	M_TP_STRIKEN   	char	20			0				0
	M_TP_TRADER    	char	10			0				0
	M_TP_TRNTYPE   	char	30			0				0
	M_TP_TYPE      	char	10			0				0
	M_TP_TYPO      	char	20			0				0
	M_TP_UQTY      	char	10			0				0
	M_TP_UQTYEQ    	char	10			0				0
	M_TRN_FMLY     	char	5			0				0
	M_TRN_GRP      	char	5			0				0
	M_TRN_GTYPE    	numeric	3	3	0	0				0
	M_TRN_TYPE     	char	5			0				0
	M_TP_CMUCLB1   	char	25			0				0
	M_TP_RTNBPHS   	numeric	2	2	0	0				0
	M_TP_SECCNT    	char	15			0				0
	M_TP_SECMKTU   	char	15			0				0
	M_TP_SMRTE     	numeric	5	9	4	0				0
/// DM_TRN_IRD_PL3_REP（PL3表）
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
	M_C_CUR_PL     	char	3			0				0
	M_C_PL_DT      	char	10			0				0
	M_DATE_REP0    	datetime	8			1				0
	M_DATE_REP1    	datetime	8			1				0
	M_DATE_REP2    	datetime	8			1				0
	M_H_ACCCPN     	numeric	11	24	4	0				0
	M_H_MPCLEAN    	numeric	11	23	4	0				0
	M_INSTRUMENT   	char	20			0				0
	M_INTVAL2      	numeric	9	19	5	0				0
	M_MP_R2        	numeric	6	11	4	0				0
	M_MP_RF1       	numeric	6	11	4	0				0
	M_MP_RF2       	numeric	6	11	4	0				0
	M_MP_SPOT2     	numeric	8	16	6	0				0
	M_MP_SPOT22    	numeric	8	16	6	0				0
	M_MP_SPOTC2    	numeric	8	16	11	0				0
	M_MP_SWAP2     	numeric	6	12	4	0				0
	M_MP_UDDRP11   	numeric	5	9	4	0				0
	M_MP_UDDRP12   	numeric	5	9	4	0				0
	M_MP_UDDRP21   	numeric	5	9	4	0				0
	M_MP_UDDRP22   	numeric	5	9	4	0				0
	M_MP_UPRC2     	numeric	10	21	11	0				0
	M_MP_VOL1      	numeric	5	9	4	0				0
	M_MP_VOL2      	numeric	5	9	4	0				0
EOD date	M_MRPL_DATE    	datetime	8			1				0
	M_MRPL_ONB     	numeric	6	10	0	0				0
	M_MX_PFOLIO    	char	16			0				0
	M_MX_REF_JOB   	numeric	6	10	0	0				0
Current (dead )trade number	M_NB           	numeric	6	10	0	0				0
	M_NB_EXT       	numeric	6	10	0	0				0
	M_NB_INIT      	numeric	6	10	0	0				0
	M_PLIRDACS12   	numeric	8	16	2	0				0
	M_PLIRDACS22   	numeric	8	16	2	0				0
	M_PLIRDCUR1    	char	3			0				0
	M_PLIRDCUR2    	char	3			0				0
	M_PLIRDFIX1    	char	1			0				0
	M_PLIRDFIX2    	char	1			0				0
	M_PLIRDFPV10   	numeric	8	16	2	0				0
	M_PLIRDFPV11   	numeric	8	16	2	0				0
	M_PLIRDFPV12   	numeric	8	16	2	0				0
	M_PLIRDFPV20   	numeric	8	16	2	0				0
	M_PLIRDFPV21   	numeric	8	16	2	0				0
	M_PLIRDFPV22   	numeric	8	16	2	0				0
	M_PLIRDNFC12   	numeric	8	16	2	0				0
	M_PLIRDNFC22   	numeric	8	16	2	0				0
	M_PL_CGR1      	numeric	11	23	6	0				0
	M_PL_CGR2      	numeric	11	23	6	0				0
	M_PL_CGRA2     	numeric	11	23	6	0				0
	M_PL_CGU1      	numeric	11	23	6	0				0
	M_PL_CGU2      	numeric	11	23	6	0				0
	M_PL_CGUA2     	numeric	11	23	6	0				0
	M_PL_CSFI2     	numeric	11	23	6	0				0
	M_PL_CSNF2     	numeric	11	23	6	0				0
	M_PL_CSNFCP2   	numeric	11	23	6	0				0
	M_PL_CSNFRV2   	numeric	11	23	6	0				0
	M_PL_FE2       	numeric	11	23	6	0				0
	M_PL_FEFI2     	numeric	11	23	6	0				0
	M_PL_FMV2      	numeric	11	23	6	0				0
	M_PL_FPFCP2    	numeric	11	23	6	0				0
	M_PL_FPFRV2    	numeric	11	23	6	0				0
	M_PL_FPNFCP2   	numeric	11	23	6	0				0
	M_PL_FPNFRV2   	numeric	11	23	6	0				0
	M_PL_FTFI1     	numeric	11	23	6	0				0
	M_PL_FTFI2     	numeric	11	23	6	0				0
	M_PL_INSCUR    	char	20			0				0
	M_PL_KEY1      	char	30			0				0
	M_PL_MVFCP2    	numeric	11	23	6	0				0
	M_PL_MVFRV2    	numeric	11	23	6	0				0
	M_PL_MVNFCP2   	numeric	11	23	6	0				0
	M_PL_NFMV2     	numeric	11	23	6	0				0
	M_PL_RVR2      	numeric	11	23	6	0				0
	M_PL_RVU2      	numeric	11	23	6	0				0
	M_PR_CTYACR2   	numeric	8	16	2	0				0
"有些CUR trade不是有好几个leg吗,该field表示当前trade是第几个leg。
"	M_QTY_INDEX    	numeric	3	3	0	0				0
	M_REF_DATA     	numeric	6	10	0	0				0
	M_RSKSECTION   	char	20			0				0
	M_TP_ACCCPN    	numeric	6	12	7	0				0
	M_TP_ACCCPN2   	numeric	6	12	7	0				0
	M_TP_ACCCPND   	numeric	6	12	7	0				0
	M_TP_AE        	char	1			0				0
	M_TP_BOSGN     	char	10			0				0
	M_TP_BROKER0   	char	15			0				0
	M_TP_BROKER1   	char	15			0				0
	M_TP_BROKER2   	char	15			0				0
	M_TP_BROKER3   	char	15			0				0
	M_TP_BUY       	char	1			0				0
	M_TP_BUY_E     	char	1			0				0
	M_TP_CMDFYS0   	char	20			0				0
	M_TP_CMDFYS1   	char	20			0				0
	M_TP_CMGQL00   	numeric	9	18	4	0				0
	M_TP_CMGQTY0   	numeric	9	18	4	0				0
	M_TP_CMGSF0    	datetime	8			1				0
	M_TP_CMGSF1    	datetime	8			1				0
	M_TP_CMIQC0    	char	3			0				0
	M_TP_CMIQU0    	char	20			0				0
	M_TP_CMLDIR0   	char	3			0				0
	M_TP_CMLDIR1   	char	3			0				0
	M_TP_CMLQU0    	char	20			0				0
	M_TP_CMUCLB0   	char	25			0				0
	M_TP_CMULAB0   	char	25			0				0
	M_TP_CMULAB1   	char	25			0				0
	M_TP_CMUQ0     	char	20			0				0
	M_TP_CMUQ1     	char	20			0				0
	M_TP_CMUQU0    	char	20			0				0
	M_TP_CMUQU1    	char	20			0				0
	M_TP_CNTRP     	char	15			0				0
	M_TP_CNTRPCY   	char	30			0				0
	M_TP_CNTRPFN   	char	255			0				0
	M_TP_CNTRPLB   	char	50			0				0
	M_TP_CP        	char	1			0				0
	M_TP_CREATOR   	numeric	6	10	0	0				0
	M_TP_CUTOFF    	char	10			0				0
	M_TP_DELIVER   	char	20			0				0
	M_TP_DIGPAY    	numeric	8	16	2	0				0
	M_TP_DIGPAYC   	char	3			0				0
The current trade 's Expired date	M_TP_DTEEXP    	datetime	8			1				0
	M_TP_DTEFLWF   	datetime	8			1				0
	M_TP_DTEFST    	datetime	8			1				0
	M_TP_DTEFXGF   	datetime	8			1				0
	M_TP_DTEFXGL   	datetime	8			1				0
	M_TP_DTELBL    	char	16			0				0
	M_TP_DTELST    	datetime	8			1				0
	M_TP_DTEPMT    	datetime	8			1				0
	M_TP_DTEPMT2   	datetime	8			1				0
The system date when trade is  Proceeing the Trade	M_TP_DTESYS    	datetime	8			1				0
trade date 	M_TP_DTETRN    	datetime	8			1				0
	M_TP_DTEUND    	datetime	8			1				0
	M_TP_DVCS      	char	1			0				0
	M_TP_ENTITY    	char	10			0				0
	M_TP_FEE0      	numeric	8	16	3	0				0
	M_TP_FEECUR0   	char	3			0				0
	M_TP_FEECUR1   	char	3			0				0
	M_TP_FEECUR2   	char	3			0				0
	M_TP_FEECUR3   	char	3			0				0
	M_TP_FXBASE    	char	3			0				0
	M_TP_FXBRSTL   	char	1			0				0
	M_TP_FXBRWED   	datetime	8			1				0
	M_TP_FXBRWSD   	datetime	8			1				0
	M_TP_FXCTPDE   	char	3			0				0
	M_TP_FXCTPFF   	numeric	6	10	0	0				0
	M_TP_FXCTPNU   	char	3			0				0
	M_TP_FXCTSFF   	numeric	8	15	7	0				0
	M_TP_FXQTOCU   	char	3			0				0
	M_TP_FXUND     	char	3			0				0
	M_TP_INITSD    	char	1			0				0
	M_TP_INT       	char	1			0				0
	M_TP_IPAY      	numeric	8	16	2	0				0
	M_TP_IPAYCUR   	char	3			0				0
	M_TP_IPAYDTE   	datetime	8			1				0
	M_TP_IQTY      	numeric	11	24	8	0				0
	M_TP_LQTY1     	numeric	11	24	8	0				0
	M_TP_LQTY2     	numeric	11	24	8	0				0
	M_TP_LQTY30    	numeric	11	24	8	0				0
	M_TP_LQTY32    	numeric	11	24	8	0				0
	M_TP_LQTYS1    	numeric	11	24	8	0				0
	M_TP_LQTYS2    	numeric	11	24	8	0				0
	M_TP_MKTNDX    	char	40			0				0
	M_TP_MOPCRT    	numeric	2	1	0	0				0
	M_TP_MOPCRTD   	datetime	8			1				0
	M_TP_MOPCRTL   	char	10			0				0
	M_TP_MOPCRTS   	char	5			0				0
	M_TP_MOPLST    	numeric	2	1	0	0				0
	M_TP_MOPLSTD   	datetime	8			1				0
Market operation(What has been done on the current trade)	M_TP_MOPLSTL   	char	10			0				0
	M_TP_NBLTI     	numeric	6	10	0	0				0
	M_TP_NOMCUR    	char	3			0				0
	M_TP_NOMINAL   	numeric	9	19	2	0				0
	M_TP_OBAR1     	numeric	8	16	6	0				0
	M_TP_OBAR2     	numeric	8	16	6	0				0
	M_TP_OBARTYP   	char	10			0				0
	M_TP_PAYCUR    	char	3			0				0
	M_TP_PFOLIO    	char	15			0				0
	M_TP_PRICE     	numeric	12	25	8	0				0
	M_TP_PRICED    	numeric	12	25	8	0				0
	M_TP_QTYEQ     	numeric	11	24	8	0				0
	M_TP_QTYEQS    	numeric	11	24	8	0				0
	M_TP_RTACR02   	numeric	8	16	2	0				0
	M_TP_RTACR12   	numeric	8	16	2	0				0
	M_TP_RTAMC01   	numeric	8	16	2	0				0
	M_TP_RTAMC02   	numeric	8	16	2	0				0
	M_TP_RTAMO     	char	20			0				0
	M_TP_RTCAP0    	numeric	9	19	2	0				0
	M_TP_RTCAP1    	numeric	9	19	2	0				0
	M_TP_RTCCP02   	numeric	12	25	8	0				0
	M_TP_RTCCP12   	numeric	12	25	8	0				0
	M_TP_RTCUR0    	char	3			0				0
	M_TP_RTCUR1    	char	3			0				0
	M_TP_RTDXC02   	datetime	8			1				0
	M_TP_RTDXC12   	datetime	8			1				0
	M_TP_RTDXG02   	datetime	8			1				0
	M_TP_RTDXG12   	datetime	8			1				0
	M_TP_RTDXP02   	datetime	8			1				0
	M_TP_RTDXP12   	datetime	8			1				0
	M_TP_RTFV0     	char	1			0				0
	M_TP_RTFV1     	char	1			0				0
	M_TP_RTFXC02   	numeric	9	19	6	0				0
	M_TP_RTFXC12   	numeric	9	19	6	0				0
	M_TP_RTFXSPT   	numeric	8	16	4	0				0
	M_TP_RTMAT0    	datetime	8			1				0
	M_TP_RTMAT1    	datetime	8			1				0
	M_TP_RTMBDC0   	char	10			0				0
	M_TP_RTMBDC1   	char	10			0				0
	M_TP_RTMCLG0   	char	10			0				0
	M_TP_RTMCNV0   	char	15			0				0
	M_TP_RTMCNV1   	char	15			0				0
	M_TP_RTMFRC0   	char	15			0				0
	M_TP_RTMFRC1   	char	15			0				0
	M_TP_RTMFRP0   	char	15			0				0
	M_TP_RTMFRP1   	char	15			0				0
	M_TP_RTMMRG0   	numeric	5	9	4	0				0
	M_TP_RTMMRG1   	numeric	5	9	4	0				0
	M_TP_RTMNDX0   	char	20			0				0
	M_TP_RTMNDX1   	char	20			0				0
	M_TP_RTMRTE0   	numeric	9	19	6	0				0
	M_TP_RTMRTE1   	numeric	9	19	6	0				0
	M_TP_RTPR0     	char	1			0				0
	M_TP_RTPR1     	char	1			0				0
	M_TP_RTRFCT0   	numeric	5	9	4	0				0
	M_TP_RTRFCT1   	numeric	5	9	4	0				0
	M_TP_RTVLC02   	numeric	9	19	6	0				0
	M_TP_RTVLC12   	numeric	9	19	6	0				0
	M_TP_RTXCHC    	char	1			0				0
	M_TP_RTXCHF    	char	1			0				0
	M_TP_RTXCHI    	char	1			0				0
	M_TP_SECCOD    	char	15			0				0
	M_TP_SECISSU   	char	40			0				0
	M_TP_SPOTN     	char	20			0				0
	M_TP_SPTFWD    	char	1			0				0
	M_TP_STATUS0   	char	10			0				0
	M_TP_STATUS1   	char	10			0				0
M_TP_STATUS2 = ''  means trade  'DEAD'	M_TP_STATUS2   	char	10			0				0
	M_TP_STRIKE    	numeric	9	18	8	0				0
	M_TP_STRIKE2   	numeric	9	18	8	0				0
交易策略（如Breakouts，Retracements，Novation）	M_TP_STRTGY    	char	15			0				0
	M_TP_TIMSYS    	char	8			0				0
	M_TP_TRADER    	char	10			0				0
	M_TP_TYPO      	char	20			0				0
	M_TP_UQTY      	char	10			0				0
	M_TP_UQTYEQ    	char	10			0				0
	M_TP_VALSTAT   	char	4			0				0
	M_TRN_FMLY     	char	5			0				0
	M_TRN_GRP      	char	5			0				0
	M_TRN_GTYPE    	numeric	3	3	0	0				0
	M_TRN_TYPE     	char	5			0				0
///DM_UDF_IRD_REP（UDF表）
	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity			Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity	
----------------	--------------	------------------------------	---------	-------	--------	--------	---------------------------	------------	-------------------	-----------			----------------	--------------	------------------------------	---------	-------	--------	--------	---------------------------	------------	-------------------	-----------
???? Null	M_AAG_BNDRY	char	1			0				0			M_AAG_BNDRY	char	1			0				0	
0	M_AAG_PERIOD	numeric	3	4	0	0				0			M_AAG_PERIOD	numeric	3	4	0	0				0	
null	M_AAG_TRNCHE	char	15			0				0			M_AAG_TRNCHE	char	15			0				0	
	M_BOOK_USER	char	10			1				0			M_BOOK_USER	char	10			1				0	
N/Y	M_CFTC_BFH	char	1			0				0			M_CFTC_BFH	char	1			0				0	
	M_CNTP_SNAME	char	15			0				0			M_CNTP_SNAME	char	15			0				0	
	M_CONFO_ID	numeric	6	10	0	0				0			M_CONFO_ID	numeric	6	10	0	0				0	
	M_CSA_EXCLUD	char	2			0				0			M_CSA_EXCLUD	char	2			0				0	
	M_CUSIP	char	50			1				0			M_CUSIP	char	50			1				0	
	M_EXT_REF_NB	char	40			1				0			M_EXT_REF_NB	char	40			1				0	
	M_IDENTITY	numeric	5	9	0	0				1			M_IDENTITY	numeric	5	9	0	0				1	
	M_IND_AMTCCY	char	3			0				0			M_IND_AMTCCY	char	3			0				0	
	M_INDEP_AMT	numeric	9	17	2	0				0			M_INDEP_AMT	numeric	9	17	2	0				0	
	M_LCH_CLR_ID	char	30			1				0			M_LCH_CLR_ID	char	30			1				0	
	M_MARKETER	char	30			0				0			M_MARKETER	char	30			0				0	
	M_MARKETER2	char	30			0	DM_UDF_COM_M_MARK_472138192			0			M_MARKETER2	char	30			0	DM_UDF_SCF_M_MARK_760139218			0	
	M_MKT_CCY	char	3			0	DM_UDF_COM_M_MKT__488138249			0			M_MKT_CCY	char	3			0	DM_UDF_SCF_M_MKT__776139275			0	
	M_MKT_IRV	numeric	4	7	0	0	DM_UDF_COM_M_MKT__504138306			0			M_MKT_IRV	numeric	4	7	0	0	DM_UDF_SCF_M_MKT__792139332			0	
	M_MX_REF_JOB	numeric	6	10	0	0				0			M_MX_REF_JOB	numeric	6	10	0	0				0	
	M_NACK_AMEND	char	30			0				0			M_NACK_AMEND	char	30			0				0	
	M_NACK_COMME	char	30			0				0			M_NACK_COMME	char	30			0				0	
	M_NACK_REASO	char	30			0				0			M_NACK_REASO	char	30			0				0	
	M_NACK_REQUE	char	30			0				0			M_NACK_REQUE	char	30			0				0	
Trade number	M_NB	numeric	6	10	0	0				0			M_NB	numeric	6	10	0	0				0	
	M_NOTIONAL	numeric	10	20	5	0				0			M_NOTIONAL	numeric	10	20	5	0				0	
	M_NOV_DT	datetime	8			1				0			M_NOV_DT	datetime	8			1				0	
	M_NOV_TRD_DT	datetime	8			1				0			M_NOV_TRD_DT	datetime	8			1				0	
	M_NOVATED	char	20			0	DM_UDF_COM_M_NOVA_253657366			0			M_NOVATED	char	20			0	DM_UDF_SCF_M_NOVA_397657879			0	
	M_OLD_CPTY	char	15			1				0			M_OLD_CPTY	char	15			1				0	
	M_OLD_NB_EXT	numeric	6	10	0	0	DM_UDF_COM_M_OLD__285657480			0			M_OLD_NB_EXT	numeric	6	10	0	0	DM_UDF_SCF_M_OLD__429657993			0	
	M_ORIG_CPTY	char	15			0	DM_UDF_COM_M_ORIG_269657423			0			M_ORIG_CPTY	char	15			0	DM_UDF_SCF_M_ORIG_413657936			0	
	M_PRICE_CAP	char	20			0				0			M_PRICE_CAP	char	20			0				0	
	M_REF_DATA	numeric	6	10	0	0				0			M_REF_DATA	numeric	6	10	0	0				0	
	M_REPGEN_DT	char	8			0				0			M_REPGEN_DT	char	8			0				0	
	M_RISK_PORT	char	15			0	DM_UDF_COM_M_RISK_520138363			0			M_RISK_PORT	char	15			0	DM_UDF_SCF_M_RISK_808139389			0	
	M_SHR_MKTR1	numeric	3	3	0	0	DM_UDF_COM_M_SHR__536138420			0			M_SHR_MKTR1	numeric	3	3	0	0	DM_UDF_SCF_M_SHR__824139446			0	
trade的类型（如 two-side）	M_DEAL_FLG	char																					
	M_SHR_MKTR2	numeric	3	3	0	0	DM_UDF_COM_M_SHR__552138477			0			M_SHR_MKTR2	numeric	3	3	0	0	DM_UDF_SCF_M_SHR__840139503			0	
Source system (Trade 来自于哪个系统)	M_SRC_SYSTEM	char	10			0				0			M_SRC_SYSTEM	char	10			0				0	
	M_STRUCT_ID	numeric	8	15	0	0				0			M_STRUCT_ID	numeric	8	15	0	0				0	
	TIMESTAMP	timestamp	8			1				0			TIMESTAMP	timestamp	8			1				0	
Expected loss	M_EXP_LOSS																						
																							
	 sp_help DM_UDF_COM_REP												 sp_help DM_UDF_SCF_REP										
																							
note	The hight fileds are not included in TAB_UDF_DEALALL_REP																						
	每条trade(无论live的还是dead的）在UDF表中都对应一条记录，记录该条trade的UDF信息																						

///TAB_MKTOP_REP(事件总表)
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
	M_BO_SGN       	char	10			0				0
	M_MX_REF_JOB   	numeric	6	10	0	0				0
Event ID, 不是trade number	M_NB           	numeric	6	10	0	0				0
	M_REF_DATA     	numeric	6	10	0	0				0
源Trade number，在该trade number 上执行了operation	M_ORIGIN_NB    	numeric	6	10	0	0				0
衍生的trade number, 由该operation  衍生出的trade number，不是所有的operation都能衍生出新的trade ,如 EXP或XIT都不能衍生出新的trade.	M_DEST_NB      	numeric	6	10	0	0				0
trader,人名	M_TRADER       	char	10			0				0
										
										
										
										
动作事件表，记录记录发生了什么，即在哪单trade上，在哪天，被谁，执行了什么operation. 该表重心在事件，而不在trade. 										
参考DM_TRN_AUD_REP表和MOP_ALL_TDY_REP表。										
///TAB_UDF_MKTOP_REP（事件总表扩展表)
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
"What was the trade changed? 如 Broker/Brokerage              
Counterparty                  
Financial （impact P&L）                    
Incorrect Source Documents    
Non-Financial   (No Impaction to P&l)              
Others (Please specify)       
Portfolio"	M_CHANGE_TYP   	char	30			0				0
对这次cancel操作做一个简短的说明，比M_C_R_CODE 列可读性更好	M_COMMENTS     	char	64			0				0
	M_CVA_USDUWD   	numeric	10	20	2	0				0
"why did the event happed?如       
BUSINESS/SYSTEM       
Credit Event        
Custom Request      
Dealer  Error        
EarlyRisk_Prime_Bkr 
EarlyRisk_Reaffirmed
EarlyRisk_Rejected  
LCH Backloading     
NOVATION                      
STEP_OUT_FULL           
Sales/Marketer Error
Trader Error            
others"	M_C_R_CODE     	char	20			0				0
	M_FVA_USDUWD   	numeric	10	20	2	0				0
	M_MKT1_SHUWD   	numeric	3	3	0	0				0
	M_MKT2_SHUWD   	numeric	3	3	0	0				0
	M_MKTER1_UWD   	char	30			0				0
	M_MKTER2_UWD   	char	30			0				0
	M_MKT_IRVUWD   	numeric	6	10	0	0				0
外键	M_MX_REF_JOB   	numeric	6	10	0	0				0
Event id (事件id),不是trade number	M_NB           	numeric	6	10	0	0				0
外键	M_REF_DATA     	numeric	6	10	0	0				0
人名	M_REQUESTOR    	char	30			0				0
	M_RK_PORTUWD   	char	15			0				0
Current trade's source system 	M_SRC_SYSTEM   	char	10			0				0
										
Note: 										
这张表事件总表(TAB_MKTOP_REP)的扩展表，记录发生该事件的原因，目前只用来记录cancel 或 cancel &reissue 的operation的其他补充信息。										


///DM_TRN_AUD_REP（trade-事件流水表）
description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity							
	TIMESTAMP      	timestamp	8			1				0							
	M_IDENTITY     	numeric	5	9	0	0				1							
"
CNCL  // Cancel market operations: a ticket can be printed at this level for audit purposes.   Generate eventID,stored in PL3 
DEL  // Delete trade ,不会产生eventID,被delete trade number 不会存到PL1 & PL3表中。 但一单新的trade(被delete的trade的新变异)会被重新插入PL1?   
*EXP  // Expiry of a trade , Generate eventID, stored in PL3        
*EXR  // exercise: a warning message can be issued for the attention of BO users when a trader exercises an option, for example.   Generate eventID, stored in PL3.     
INS    // Insert trade   ,不会产生 enventID, stored in PL1 
INSCNCL  // Insert cancel????    ，不会产生eventID， trade number 不会存到PL3/PL1表中
MOD   // Modify trade         
*NET   // Netting of securities or listed products(MOP)  ，have eventID, stored in PL3
*RPL    // Restructure(MOP):contractual re-negotiation of trade , Generate eventiD, stored 2 Items  in PL3       
RPL_DEL  // Cancel of trades(MOP): trade is deleted but remains in the database for audit purposes. , And 2 items(M_ACT_NB2=0/1) only for CUR trade sotred in DM_TRN_AUD_REP and  Generate eventID, stored in PL3. As of  the cancel Date the deal never has no more effect whatever on book positions or PL.
* RPL_MOD // Cancel and re-issue(MOP): correction of an error.   Generate eventID, stored in PL3
XIT  // Early termination    , Generate eventID, store 2 items in PL3 when trade is a CUR trade . 
(* Standard operation, others stand advanced operations)"	M_ACTION       	char	15			0				0							
	M_ACT_COM0     	char	100			0				0							
	M_ACT_COM1     	char	100			0				0							
	M_ACT_COM2     	char	100			0				0							
事件Trade Number	M_ACT_NB0      	numeric	6	10	0	0				0							
标记curent trade的源头（或保存生成M_ACT_NB0 trade的Event ID 或 保存M_ACT_NB0 trade所对应的dead leg的 trade ID） ， 见下面的例子去理解(同一个事件eventID可能会引起两个或更多操作如EXR事件之后会伴随着一个 CNCL事件；一个PRL事件之后会伴随两个CNCL事件同时发生)	M_ACT_NB1      	numeric	9	17	6	0				0							
0 or 1，与leg有关(for Swaps you would have leg0 and leg1)	M_ACT_NB2      	numeric	9	17	6	0				0							
	M_BO_FO        	char	10			0				0							
Murex date（roll date 时，roll的就是它）, 一定大于或等于M_DATE_CMP(Actural date)	M_DATE         	datetime	8			1				0							
Actual date (系统日期) 	M_DATE_CMP     	datetime	8			1				0							
	M_ID           	numeric	6	10	0	0				0							
	M_MX_PFOLIO    	char	16			0				0							
	M_MX_REF_JOB   	numeric	6	10	0	0				0							
	M_REF_DATA     	numeric	6	10	0	0				0							
Actual time(Second), Max(M_TIME_CMP) = 3600*24	M_TIME_CMP     	numeric	5	8	0	0				0							
	M_TIME_ZONE    	numeric	2	2	0	0				0							
i.e. TRANSACTION	M_TYPE         	char	15			0				0							
	M_USR_DESK     	char	10			0				0							
	M_USR_GROUP    	char	10			0				0							
	M_USR_NAME     	char	10			0				0							
																	
当有人对trade做了任何action(operation)都会触发一条记录插入到该表。记录每条trade的事件流水记录，																	
该表重心在trade，而不在事件，记录所有发生operation的trade的前身(被 cancel的trade或event ID)																	
参考TAB_MKTOP_REP表。																	
																	
																	
例1：Trader手动输入了一单新trade(trade 1),被记录为：																	
	M_ACTION       	M_ACT_NB0      	M_ACT_NB1      	M_ACT_NB2										PL1 table		PL3 table	
	INR	trade1 ID	0	0										M_NB	M_TP_PFOLIO	M_NB	M_TP_PFOLIO
														trade1 ID	Port_1		
																	
例2：Trader在刚刚的新trade(trade 1)上执行了cancel&reissue操作（Event 2），同时再插入另一单新的trade(trade 2),该过程被记录为：									MOP_ALL table:					PL1 table		PL3 table	
	M_ACTION       	M_ACT_NB0      	M_ACT_NB1      	M_ACT_NB2					M_NB	M_DEST_NB	FINANCIAL			M_NB	M_TP_PFOLIO	M_NB	M_TP_PFOLIO
	PRL_MOD	trade1 ID	event2 ID	0					trade1 ID	trade2 ID	TP_PORT,Port_1,Port_2			trade2 ID	Port_2	trade1 ID	Port_1
	INR	trade2 ID	trade1 ID	0													
																	
例3：Trader在刚刚的reissue生成的新trade(trade 2)上又执行了cancel&reissue操作（Event 3），同时再插入一单新的trade(trade 3),该过程被记录为：									MOP_ALL table:					PL1 table		PL3 table	
	M_ACTION       	M_ACT_NB0      	M_ACT_NB1      	M_ACT_NB2					M_NB	M_DEST_NB	FINANCIAL			M_NB	M_TP_PFOLIO	M_NB	M_TP_PFOLIO
	PRL_MOD	trade2 ID	event3 ID	0					trade1 ID	trade2 ID	TP_PORT,Port_1,Port_2			trade3 ID	Port_3	trade1 ID	Port_1
	INR	trade3 ID	trade2 ID	0					trade2 ID	trade3 ID	TP_PORT,Port_1,Port_2;TP_PORT,Port_2,Port_3;					trade2 ID	Port_2
																	
																	
																	
																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'CNCL'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'DEL'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'EXP'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'EXR'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'INS'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'INSP'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'INSCNCL'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'MOD'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'NET'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'RPL'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'RPL_DEL'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'RPL_MOD'																	
UNION ALL																	
select top 2 M_ACT_NB0,M_ACTION,M_ACT_NB1,M_ACT_NB2,M_DATE,M_DATE_CMP from DM_TRN_AUD_REP where M_ACTION = 'XIT'																	
																	
M_ACT_NB0	M_ACTION	M_ACT_NB1	M_ACT_NB2	M_DATE	M_DATE_CMP												
44268820	CNCL           	27368639	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44267349	CNCL           	27367750	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44393599	DEL            	0	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44393600	DEL            	0	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
43166385	EXP            	27441443	0	May  9 2016 12:00AM	May  7 2016 12:00AM												
43166387	EXP            	27441444	0	May  9 2016 12:00AM	May  7 2016 12:00AM												
42989607	EXR            	27442157	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
39631042	EXR            	27442158	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44298637	INS            	0	0	May  2 2016 12:00AM	Apr 29 2016 12:00AM												
44298638	INS            	0	0	May  2 2016 12:00AM	Apr 29 2016 12:00AM												
44456122	INSP           	0	0	May 12 2016 12:00AM	May 12 2016 12:00AM												
44456132	INSP           	0	0	May 12 2016 12:00AM	May 12 2016 12:00AM												
44381870	INSCNCL        	0	0	May  9 2016 12:00AM	May  6 2016 12:00AM												
44381871	INSCNCL        	0	0	May  9 2016 12:00AM	May  6 2016 12:00AM												
0	MOD            	0	0	May  2 2016 12:00AM	Apr 29 2016 12:00AM												
0	MOD            	0	0	May  2 2016 12:00AM	Apr 29 2016 12:00AM												
44298615	NET            	27386191	0	May  2 2016 12:00AM	Apr 29 2016 12:00AM												
42743852	NET            	27386191	0	May  2 2016 12:00AM	Apr 29 2016 12:00AM												
43387834	RPL            	27442990	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
43011752	RPL            	27442992	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44382979	RPL_DEL        	27442154	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44381071	RPL_DEL        	27442175	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
44381301	RPL_MOD        	27441427	0	May  9 2016 12:00AM	May  6 2016 12:00AM												
44382560	RPL_MOD        	27442146	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
43030327	XIT            	27442147	0	May  9 2016 12:00AM	May  9 2016 12:00AM												
43030330	XIT            	27442148	0	May  9 2016 12:00AM	May  9 2016 12:00AM												


///MOP_ALL_TDY_REP(trade-C&A流水表)
description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
发生cancel或cancel&reissue动作的trade number	M_NB           	numeric	6	10	0	0				0
被reissue的trade number	M_DEST_NB      	numeric	6	10	0	0				0
发生cancel或cancel&reissue动作的trade 的交易日期	M_DATE_CMP     	datetime	8			0				0
Actural system date	M_DATE         	datetime	8			0				0
执行了该动作的user的名字	M_USR_NAME     	char	10			0				0
当前trade(M_NB标识的trade)的cancel&reissue Event ID	M_ACT_NB1      	numeric	9	17	0	0				0
0 OR 1	M_ACT_NB2      	numeric	9	17	0	0				0
0 OR 1	M_C_ACT_NB2    	numeric	9	17	0	0				0
如果当前的cancel或cancel&reissue动作影响了该trade的P&L, 必要的信息将被记录在该列	FINANCIAL      	varchar	512			0				0
如果当前的cancel或cancel&reissue动作不影响该trade的P&L, 相关的信息记录该列	NON_FIN        	varchar	512			0				0
user  group	M_USR_GROUP    	char	30			1				0
										
										
这张以trade 动作表DM_TRN_AUD_REP(trade-事件总表)为数据源表，通过store procedure（scb_populate_mop_all.sql）提取出当天的关于cancel 和cancel&reissue动作的信息。将DEAD leg和LIVE leg 的信息体现在同一条记录中。										
该表的重心在事件而不在trade										

///TAB_TRN_HDR1_REP
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP	timestamp	8			1				0
	M_IDENTITY	numeric	5	9	0	0				1
	M_BCOMMENT0	char	30			0				0
	M_BCOMMENT1	char	30			0				0
	M_BCOMMENT2	char	30			0				0
	M_BRW_ODPL	char	20			0				0
(case when M_COMMENT_BS='S' then a.M_SPFOLIO else a.M_BPFOLIO end) as Portfolio	M_COMMENT_BS	char	1			0				0
	M_MX_REF_JOB	numeric	6	10	0	0				0
Trade number	M_NB	numeric	6	10	0	0				0
System EOD	M_OPT_MOPLSD	datetime	8			1				0
	M_OPT_MOPLST	datetime	8			1				0
	M_REF_DATA	numeric	6	10	0	0				0
	M_SCOMMENT0	char	30			0				0
	M_SCOMMENT1	char	30			0				0
	M_SCOMMENT2	char	30			0				0
	M_SYS_DATE	datetime	8			1				0
Trade status	M_TRN_STATUS	char	10			0				0
Event number(TAB_MKTOP_REP.M_NB)	M_OPT_MOPNB	numeric	6	10	0	0				0
	M_MOP_CREAT	numeric	2	1	0	0				0
	M_BRW_NOM2	numeric	11	24	8	0				0
	M_BRW_NOM1	numeric	11	24	8	0				0
Linked trades' reference number 	M_LTI_NB	numeric	6	10	0	0				0
	M_RPL_DATE1	datetime	8			1				0
	M_RPL_AMT	numeric	8	16	2	0				0
	M_RPL_DATE2	datetime	8			1				0
	M_TRN_EXP	datetime	8			1				0
	M_IRV_AMT	numeric	8	16	2	0	TAB_TRN_HD_M_IRV__392137907			0
	M_BRW_NOMU1	char	3			0	TAB_TRN_HD_M_BRW__1078252561			0
	M_BRW_NOMU2	char	3			0	TAB_TRN_HD_M_BRW__1094252618			0
	M_BRW_RTE1	numeric	10	21	8	0	TAB_TRN_HD_M_BRW__1110252675			0
	M_BRW_RTE2	numeric	6	12	4	0	TAB_TRN_HD_M_BRW__1126252732			0
	M_DTE_AMD	datetime	8			1				0
	M_INSTRUMENT	char	20			0	TAB_TRN_HD_M_INST_1142252789			0
	M_CREATOR	numeric	6	10	0	0	TAB_TRN_HD_M_CREA_1158252846			0
										
										
linked trade main table , its source table is TRN_HDR_DBF										
A linked trade has a reference number										
///MKT_OPT_DB header
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
Market ID  (Not trade number)	M_NB           	numeric	6	10	0	0				0
Current status in market operation workflow	M_VAL_STATUS   	char	4			0				0
	M_TRN_FMLY     	char	5			0				0
	M_TRN_GRP      	char	5			0				0
	M_TRN_TYPE     	char	5			0				0
	M_INSTRUMENT   	char	20			0				0
	M_RSKSECTION   	char	20			0				0
	M_PL_INSCUR    	char	3			0				0
	M_PL_KEY1      	char	30			0				0
	M_MKT_INDEX    	char	40			0				0
	M_TYPE         	char	10			0				0
	M_TYPE_SUB     	char	10			0				0
Murex date	M_DATE         	datetime	8			1				0
murex time < 3600s*24	M_TIME         	numeric	6	10	0	0				0
actual date	M_SYS_DATE     	datetime	8			1				0
	M_AGREED_OP    	char	1			0				0
	M_ORIGIN       	char	15			0				0
Original trade number (i.e. canceled trade)	M_ORIGIN_NB    	numeric	6	10	0	0				0
	M_DEST         	char	15			0				0
new trade number (i.e. reissued trade)	M_DEST_NB      	numeric	6	10	0	0				0
	M_NFAMOUNT     	numeric	8	16	2	0				0
	M_CURRENCY     	char	3			0				0
	M_NFAMOUNTD    	datetime	8			1				0
	M_O_AMOUNT     	numeric	8	16	2	0				0
	M_S_SPOT       	numeric	6	10	5	0				0
	M_S_PRICE      	numeric	10	21	8	0				0
	M_BO_SGN       	char	10			0				0
	M_BO_CMT       	char	10			0				0
	M_BO_CNF       	char	10			0				0
	M_AREA_CODE    	numeric	2	2	0	0				0
	M_TIME_ZONE    	numeric	2	2	0	0				0
	M_TIME_ZONEA   	numeric	2	2	0	0				0
	M_TRADER       	char	10			0				0
	M_HDG_SOLD     	numeric	2	1	0	0				0
	M_PURGE_STS    	numeric	2	1	0	0				0
	M_PURGE_DATE   	datetime	8			1				0
	M_PURGE_GRP    	numeric	6	10	0	0				0
	M_COMMENT      	char	10			0				0
	M_LAT          	char	1			0				0
valid date	M_VAL_DATE     	datetime	8			1				0

///TRMKTOP_DBF body
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
MKT_OPT_DB.M_NB           	M_MKT_NB       	numeric	6	10	0	0				0
original Trader number	M_TRN_NB       	numeric	6	10	0	0				0
new reissue trade number	M_BROTHER      	numeric	6	10	0	0				0
	M_ACCOUNT      	char	15			0				0
	M_QTY_INDEX    	numeric	3	3	0	0				0
initial quantity	M_IQTY         	numeric	11	24	8	0				0
Market operation quantity	M_QTY          	numeric	11	24	8	0				0
	M_LSTTRN_NB    	numeric	2	1	0	0				0
	M_NETMAXSTL    	datetime	8			1				0
	M_NET_BR_NB    	numeric	6	10	0	0				0
	M_NET_QTY      	numeric	11	24	8	0				0


///TRN_HDRF_DBF  fees table
Description	Column_name	Type	Length	Prec	Scale	Nulls	Default_name	Rule_name	Access_Rule_name	Identity
	TIMESTAMP      	timestamp	8			1				0
	M_IDENTITY     	numeric	5	9	0	0				1
	M_FLOW_FMLY    	char	10			0				0
	M_NB           	numeric	6	10	0	0				0
	M_NB_MOP       	numeric	6	10	0	0				0
	M_QTY_INDEX    	numeric	3	3	0	0				0
	M_STL_TYPE     	numeric	2	2	0	0				0
	M_FLOW_TYPE    	char	10			0				0
	M_FLOW_TPNB    	numeric	2	2	0	0				0
	M_FLOW_TPT0    	char	10			0				0
	M_FLOW_TPT1    	char	10			0				0
	M_FLOW_TPT2    	char	10			0				0
	M_FLOW_TPT3    	char	10			0				0
	M_FLOW_TPT4    	char	10			0				0
	M_FLOW_TPL0    	char	10			0				0
	M_FLOW_TPL1    	char	10			0				0
	M_FLOW_TPL2    	char	10			0				0
	M_FLOW_TPL3    	char	10			0				0
	M_FLOW_TPL4    	char	10			0				0
	M_DATE         	datetime	8			1				0
	M_SDATE_FLAG   	numeric	2	1	0	0				0
	M_SDATE        	datetime	8			1				0
	M_AMOUNT       	numeric	9	19	3	0				0
	M_CURRENCY     	char	3			0				0
	M_COMMENT      	char	100			0				0
	M_CTP          	char	15			0				0
	M_FIX_DET      	numeric	2	1	0	0				0
	M_FIX_QTYNX0   	numeric	3	3	0	0				0
	M_FIX_QTYNX1   	numeric	3	3	0	0				0
	M_FIX_QTYNX2   	numeric	3	3	0	0				0
	M_FIX_PHASE0   	numeric	2	2	0	0				0
	M_FIX_PHASE1   	numeric	2	2	0	0				0
	M_FIX_PHASE2   	numeric	2	2	0	0				0
	M_FIX_LEG0     	numeric	2	2	0	0				0
	M_FIX_LEG1     	numeric	2	2	0	0				0
	M_FIX_LEG2     	numeric	2	2	0	0				0
	M_FIX_NDX0     	char	20			0				0
	M_FIX_NDX1     	char	20			0				0
	M_FIX_NDX2     	char	20			0				0
	M_FIX_NDXSL0   	char	20			0				0
	M_FIX_NDXSL1   	char	20			0				0
	M_FIX_NDXSL2   	char	20			0				0
	M_FIX_NDXST0   	numeric	3	3	0	0				0
	M_FIX_NDXST1   	numeric	3	3	0	0				0
	M_FIX_NDXST2   	numeric	3	3	0	0				0
	M_FIX_DTE0     	datetime	8			1				0
	M_FIX_DTE1     	datetime	8			1				0
	M_FIX_DTE2     	datetime	8			1				0
	M_FIX_DTECF0   	datetime	8			1				0
	M_FIX_DTECF1   	datetime	8			1				0
	M_FIX_DTECF2   	datetime	8			1				0
	M_FIX_DTECL0   	datetime	8			1				0
	M_FIX_DTECL1   	datetime	8			1				0
	M_FIX_DTECL2   	datetime	8			1				0
	M_REF_NOS      	numeric	6	10	0	0				0
	M_REF_VOS      	numeric	6	10	0	0				0
	M_FIX_RLEG0    	numeric	2	2	0	0				0
	M_FIX_RLEG1    	numeric	2	2	0	0				0
	M_FIX_RLEG2    	numeric	2	2	0	0				0
	M_PER_STRK     	numeric	9	19	3	0				0
	M_LEG_IS_RCV   	numeric	2	2	0	0				0
	M_REFERENCE    	numeric	6	10	0	0				0
	M_ORG_REF      	numeric	6	10	0	0				0
	M_ORG_FID      	numeric	3	3	0	0				0
	M_ORG_TRNNB    	numeric	6	10	0	0				0
	M_GLB_TYPO     	char	10			0				0
	M_PER_RATE     	numeric	8	16	6	0				0
	M_PER_NOM      	numeric	9	19	3	0				0
	M_PER_NUNIT0   	char	10			0				0
	M_PER_NUNIT1   	char	10			0				0
	M_PER_NUNIT2   	char	10			0				0
	M_PER_FCONV0   	char	15			0				0
	M_PER_FCONV1   	char	15			0				0
	M_PER_FCONV2   	char	15			0				0
	M_PER_NDX0     	char	20			0				0
	M_PER_NDX1     	char	20			0				0
	M_PER_NDX2     	char	20			0				0
	M_PER_MRG0     	numeric	6	10	5	0				0
	M_PER_MRG1     	numeric	6	10	5	0				0
	M_PER_MRG2     	numeric	6	10	5	0				0
	M_FRM_DEN      	numeric	6	10	4	0				0
	M_FRM_NUM      	numeric	6	10	4	0				0
	M_SE_CODE      	char	15			0				0
	M_SE_LABEL     	char	15			0				0
	M_SE_MARKET    	char	10			0				0
	M_SE_QTY       	numeric	11	22	8	0				0
	M_SE_PRICE     	numeric	11	22	12	0				0
	M_TRD_DATE     	datetime	8			1				0
	M_AMORT_DATE   	datetime	8			1				0
	M_STTAMT_DAT   	datetime	8			1				0


///note
## Common Database ##																							
                TAB_ALLTRNRP_PL1_REP:  you can got the common trade info from the table, ,which will never be used after Rubicon Project completion.																							
                TAB_ALLTRNRP_PL2_REP:  Due to Sybase table constrain (one table only include 100 fields), we create the table as supplement,,which will never be used after Rubicon Project completion.																							
                TAB_TRN_HDR1_REP: trade header REP table																							
                DM_TRN_USRD_REP: user info REP table																							
                TAB_UDF_DEALALL_REP:  ALL product UDF REP table,which will never be used after Rubicon Project completion.																							
                DM_UDF_$PRODUCTION_REP(DM_UDF_COM_REP) : each product UDF REP table																							
               TABLE#DATA#DEALCURR_DBF :  Financial Dabase UDF table																							
                DM_TRNRP_CST1L_REP: live trade cash flow REP table																							
                DM_TRNRP_CST1D_REP: Dead trade cash flow REP table																							
                DM_COUNTERPARTY_REP: counterparty REP table																							
                TAB_SPTCV2_REP: spot (rate convert)																							
                DM_TRNRP_PL3_REP: Dead trade info																							
               TABLE#DATA#COUNTERP_DBF      Financial Dabase UDF table (Counterparty User Defined Fields)																							
																							
																							
####  Datamart DB 很重要的三张表，两个重要的公共字段																							
select distinct T1.M_TRN_FMLY   from  TAB_ALLTRNRP_PL1_REP T1										select distinct T1.M_TRN_FMLY from  TAB_ALLTRNRP_PL2_REP T1										select distinct T1.M_TRN_FMLY from  DM_TRNRP_PL3_REP T1			
---------------------------										---------------------------										---------------------------			
M_TRN_FMLY										M_TRN_FMLY										M_TRN_FMLY			
COM  										COM  										COM  			
CRD  										CRD  										CRD  			
CURR 										CURR 										CURR 			
IRD  										IRD  										IRD  			
SCF  										SCF  										SCF  			
																							
#####																							
select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL1_REP T1 where T1.M_TRN_FMLY = 'COM'										select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL2_REP T1 where T1.M_TRN_FMLY = 'COM'										select distinct T1.M_TRN_GTYPE from DM_TRNRP_PL3_REP T1 where T1.M_TRN_FMLY = 'COM'			
-----------------------------			 DMDBO.TAB_TRN_COM_PL1_REP							-----------------------------			DMDBO.TAB_TRN_COM_PL2_REP							-----------------------------			DMDBO.DM_TRN_COM_PL3_REP
M_TRN_GTYPE										M_TRN_GTYPE										M_TRN_GTYPE			
100										100										100			
101										101										101			
102										102										102			
103										103										103			
130										130										130			
131										131										131			
132										132										132			
136										136										136			
113										113										113			
																							
select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL1_REP T1 where T1.M_TRN_FMLY = 'CRD'										select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL2_REP T1 where T1.M_TRN_FMLY = 'CRD'										select distinct T1.M_TRN_GTYPE from DM_TRNRP_PL3_REP T1 where T1.M_TRN_FMLY = 'CRD'			
-----------------------------			DMDBO.TAB_TRN_CRD_PL1_REP							-----------------------------			DMDBO.TAB_TRN_CRD_PL2_REP							-----------------------------			DMDBO.DM_TRN_CRD_PL3_REP
M_TRN_GTYPE										M_TRN_GTYPE										M_TRN_GTYPE			
29										29										29			
95										95										95			
98										98										98			
122										122										122			
124										124										123			
																				124			
select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL1_REP T1 where T1.M_TRN_FMLY = 'CURR'										select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL2_REP T1 where T1.M_TRN_FMLY = 'CURR'										select distinct T1.M_TRN_GTYPE from DM_TRNRP_PL3_REP T1 where T1.M_TRN_FMLY = 'CURR'			
-----------------------------			DMDBO.TAB_TRN_CUR_PL1_REP							-----------------------------			DMDBO.TAB_TRN_CUR_PL2_REP							-----------------------------			DMDBO.DM_TRN_CUR_PL3_REP
M_TRN_GTYPE										M_TRN_GTYPE										M_TRN_GTYPE			
70										70										70			
71										71										71			
72										72										72			
74										74										74			
76										76										76			
77										77										77			
80										80										80			
82										82										82			
84										84										84			
85										85										85			
86										86										86			
87										87										87			
88										88										88			
89										89										89			
																							
																							
select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL1_REP T1 where T1.M_TRN_FMLY = 'IRD'										select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL2_REP T1 where T1.M_TRN_FMLY = 'IRD'										select distinct T1.M_TRN_GTYPE from DM_TRNRP_PL3_REP T1 where T1.M_TRN_FMLY = 'IRD'			
-----------------------------			DMDBO.TAB_TRN_IRD_PL1_REP							-----------------------------			DMDBO.TAB_TRN_IRD_PL2_REP							-----------------------------			DMDBO.DM_TRN_IRD_PL3_REP
M_TRN_GTYPE										M_TRN_GTYPE										M_TRN_GTYPE			
1										1										1			
2										2										2			
3										3										3			
4										4										4			
5										5										5			
8										8										8			
19										19										19			
20										20										20			
41										41										41			
42										42										49			
49										49										50			
50										50										63			
																							
select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL1_REP T1 where T1.M_TRN_FMLY = 'SCF'										select distinct T1.M_TRN_GTYPE from TAB_ALLTRNRP_PL2_REP T1 where T1.M_TRN_FMLY = 'SCF'										select distinct T1.M_TRN_GTYPE from DM_TRNRP_PL3_REP T1 where T1.M_TRN_FMLY = 'SCF'			
-----------------------------			DMDBO.TAB_TRN_SCF_PL1_REP							-----------------------------			DMDBO.TAB_TRN_SCF_PL2_REP							-----------------------------			DMDBO.DM_TRN_SCF_PL3_REP
M_TRN_GTYPE										M_TRN_GTYPE										M_TRN_GTYPE			
90										90										90			
																							
																							
																							
																							
																							
																							
																							
Journey shared:																							
select T1.M_IDENTITY, T1.M_ID, T1.M_FAMILY, T1.M_GROUP, T1.M_TYPE, T1.M_LABEL, T1.M_DESC, T1.M_GEN_PAY from TRN_TYPO_DBF T1  order by T1.M_ID asc																							
																							
																							
																							
																							
																							
																							
DB 中没有我想要的数据，这种情况下，我应该怎么发邮件请求Wesley他们为我刷DB dump呢？																							
[Cathy] User  booked some trade on 11 Aug , so you can get 11 Aug dump after 12 Aug.																							
																							
																							
还有个问题，你们要求他们刷DB到指定的某一天，是根据什么条件？（哪个比表的哪个字段来决定你要刷到那一天的？）																							
[Cathy] there are two sql to check the db date.																							
select M_REP_DATE_2 from DMDBO.DM_DATES_REP----------------------------mxgdb																							
select M_REP_DATE_2 from MUREXDB.ALLDATES_DBF------------mxg																							
																							
																							
																							
																							
MPX_SPOT_DBF    SPOT Rate stored in this table																							
																							
																							
select * from MPX_SPOT_DBF where M__ALIAS_='./BORATES' and M__DATE_= (select max(M__DATE_) from MPX_SPOT_DBF where M__DATE_ not in (select max(M__DATE_) from MPX_SPOT_DBF))																							
																							
																							
																							
																							
																							
Main Category of tables (Supervisor -> Group/xxx/Consistency/Template)																							
Bundle	Check on object delete																						
	Main																						
Call/Deposit	Authorize overdraft																						
Combined Portfolio	Erasable test																						
Confirmation instr.	Address																						
	Custom Information																						
	Language																						
	Master Agreement																						
Counterpart	Bank code																						
	User fields																						
Deals table	COM|ASIAN|																						
	COM|ASIAN|CLR																						
	COM|EFS|																						
	COM|FUT|																						
	COM|FWD|																						
	COM|LB|																						
	COM|OFUT|LST																						
	COM|OFUT|OTC																						
	COM|OPT|CMP																						
	COM|OPT|SMP																						
	COM|OPT|SWAP																						
	COM|SPOT|																						
	COM|SWAP|																						
	COM|SWAP|CLR																						
	CRD|CDS|																						
	CRD|CFUT|																						
	CRD|CRDIO|																						
	CRD|CRDI|																						
	CRD|EDS|																						
	CRD|FDB|																						
	CRD|FL|																						
	CRD|NDB|																						
	CRD|OASWP|																						
	CRD|OBDS|																						
	CRD|OCDO|																						
	CRD|OCDS|																						
	CRD|PCDS|																						
	CRD|RLOAN|																						
	CRD|RTRS|																						
	CRD|SCDO|																						
	CURR|FUT|FUT																						
	CURR|FXD|FXD																						
	CURR|FXD|FXDS																						
	CURR|FXD|XSW																						
	CURR|OPT|ASN																						
	CURR|OPT|BAR																						
	CURR|OPT|BAR2																						
	CURR|OPT|BOF																						
	CURR|OPT|BSK																						
	CURR|OPT|CMP																						
	CURR|OPT|FLEX																						
	CURR|OPT|KIKO																						
	CURR|OPT|LKB																						
	CURR|OPT|LST																						
	CURR|OPT|RBT																						
	CURR|OPT|RBTS																						
	CURR|OPT|SMP																						
	CURR|OPT|SMPS																						
	CURR|OPT|STRA																						
	EQD|BOND|CNV																						
	EQD|BOND|IDX																						
	EQD|COL|																						
	EQD|EQS|																						
	EQD|EQUIT|																						
	EQD|EQUIT|FWD																						
	EQD|FS|																						
	EQD|FUT|																						
	EQD|LB|																						
	EQD|OPT|ACASN																						
	EQD|OPT|ACC																						
	EQD|OPT|ACPUT																						
	EQD|OPT|ALTPN																						
	EQD|OPT|ASI																						
	EQD|OPT|AUTOC																						
	EQD|OPT|BAR																						
	EQD|OPT|BERMU																						
	EQD|OPT|CCLQT																						
	EQD|OPT|CLQT																						
	EQD|OPT|CNDVS																						
	EQD|OPT|CRAC																						
	EQD|OPT|CRBSK																						
	EQD|OPT|CRDR																						
	EQD|OPT|CRRVS																						
	EQD|OPT|DASN																						
	EQD|OPT|DBARR																						
	EQD|OPT|DBCOU																						
	EQD|OPT|DIGLB																						
	EQD|OPT|FLEX																						
	EQD|OPT|HMLY																						
	EQD|OPT|KICAL																						
	EQD|OPT|LAD																						
	EQD|OPT|LBCLQ																						
	EQD|OPT|LKB																						
	EQD|OPT|MBARR																						
	EQD|OPT|NPLN																						
	EQD|OPT|ORG																						
	EQD|OPT|OTC																						
	EQD|OPT|PICUP																						
	EQD|OPT|RAT																						
	EQD|OPT|RELAX																						
	EQD|OPT|RGACC																						
	EQD|OPT|RNBW																						
	EQD|OPT|RNBWB																						
	EQD|OPT|RVPD																						
	EQD|OPT|SWING																						
	EQD|OPT|SWP																						
	EQD|OPT|TGTAS																						
	EQD|OPT|VOLSW																						
	EQD|OPT|VSWP																						
	EQD|REPO|																						
	EQD|REPO|REPO																						
	EQD|SHS|																						
	EQD|WARNT|																						
	FIN|BSB|																						
	FIN|CFD|																						
	FIN|LB|																						
	FIN|REPO|																						
	FXD|FXD|FXD																						
	HYB|OPT|HMLY																						
	HYB|OPT|RNBW																						
	HYB|OPT|RNBWB																						
	IRD|ASWP|																						
	IRD|BOND|																						
	IRD|BOND|CALL																						
	IRD|BOND|FWD																						
	IRD|CD|																						
	IRD|CF|																						
	IRD|CS|																						
	IRD|FRA|																						
	IRD|INFLS|																						
	IRD|IRG|																						
	IRD|IRS|																						
	IRD|LB|																						
	IRD|LFUT|																						
	IRD|LN_BR|																						
	IRD|OPT|ASI																						
	IRD|OPT|BAR																						
	IRD|OPT|FLEX																						
	IRD|OPT|LAD																						
	IRD|OPT|ORG																						
	IRD|OPT|OTC																						
	IRD|OPT|RAT																						
	IRD|OSWP|																						
	IRD|REPO|																						
	IRD|REPO|REPO																						
	IRD|SFUT|																						
	IRD|TRS|																						
	IRD|WARNT|																						
	Non financial info																						
	SCF|SCF|SCF																						
Early termination	Disable fee																						
	Unrestrict XIT																						
Fixing	Audit																						
	Global																						
	Past date																						
	Present date																						
Hedge	Hedge																						
	Hedge currency																						
	Hedge entity																						
	Hedge prospective template																						
	Hedge template																						
	Hedge type																						
	Risk type																						
	Strategy code																						
	User definable																						
	VaR reports																						
Linked trade	Apply status rules on trades - Ins/Mod																						
	Apply status rules on trades - ValAction																						
	Cancel deal by deal																						
	Duplicate																						
	Inter-entity package																						
	Package structure user fields																						
	Perform market operation on single trade																						
	Unlink																						
	Validate inconsistent package																						
Manual acc. entries	Entry date																						
	Entry date >= value date and pay. date																						
	Payment date																						
	Value date																						
Market operation	Advanced Exercise																						
	Advanced early term																						
	Allocate: B2B Destination																						
	Allocate: B2B Origin																						
	Allocate: Destination																						
	Allocate: Origin																						
	Allocate: Template																						
	Assignment																						
	Cancel																						
	Cancel and reissue																						
	Close out																						
	Corporate Action																						
	Counterpart assignme																						
	Delta gap agreement																						
	Early take-up																						
	Early termination																						
	Exercise																						
	Exercise cancellable																						
	Expiry																						
	Extension																						
	Extension (multiple)																						
	FX Callable Exercise																						
	MM date roll-over																						
	Netting																						
	Nonfin info in new deal																						
	Perform at future date																						
	Prolongation																						
	Restructure																						
	Settlement event																						
	Split																						
	User Fields																						
P&L reference	Allow results modification																						
Payment entry	Comment fields																						
	Family/Group/Type																						
	Manual flow: acc. section																						
	Manual flow: entity																						
	Manual flow: trading section																						
	Settlement																						
	User fields																						
Payment table	Manual flow																						
	Movements																						
Portfolio entry	Accounting Section																						
	Entity																						
RT Deals Interface	Deals Monitor																						
	Enable/Disable Service																						
	Links Definition																						
	Purge Database																						
	Start Deals Application server																						
	Start Deals Interface Server																						
	Stop  Deals Application Server																						
	Stop  Deals Interface Server																						
	Stop All Servers																						
	Super RT User																						
Restructure	Disable P&L computation																						
	Partial restructure																						
Risk matrix	Cash & Futre Flow																						
	Delta Hedge																						
	Delta gap topography																						
	Fill smile																						
	Fixing Analysis																						
	MKTP import																						
	Market Risk Extraction																						
	Open Position Extraction																						
	Pin report																						
	Risk Matrix Extraction																						
	Screening batches																						
	Skew analysis																						
	Standard Risk Matrix																						
	Theta Anlysis																						
	Topography																						
	Trading Limit																						
	Value at Risk																						
Routing deal module	Deal fields assignment																						
Settlement	Authorize NWD flow																						
Trade entry	Accounting section																						
	Additional flows																						
	Brokerage																						
	Check counterparty existence																						
	Comment																						
	Compute premium																						
	Confirmation																						
	Counterparty																						
	Date																						
	Dest. Strategy																						
	Dest. Trader																						
	Draft																						
	External#																						
	I/E																						
	Internal Trader																						
	More (sub-screen)																						
	Payment condition																						
	Portfolio																						
	Product typology																						
	Sales																						
	Settlement																						
	Source Strategy																						
	Time																						
	Typology popup																						
	User Fields																						
Trade event	Addition																						
	Breakage																						
	CD Adjustement																						
	CD Client deposit																						
	CD Client withdrawal																						
	CD Close account																						
	CD Generic event																						
	CD Interest payment																						
	CD Interest reinvest																						
	CD Margin adjustment																						
	CD Next Pay date adj																						
	CD Pay interest																						
	CD Rate adjustment																						
	CD Sales margin adj																						
	CD Total deposit																						
	CD Total withdrawal																						
	COM Delivery event																						
	COM Volume event																						
	Closing																						
	Coll Rate repricing																						
	Credit event notice																						
	Decrease																						
	Default																						
	Deletion																						
	FS additional. subs.																						
	FS close subs.																						
	FS partial close																						
	Fictive default																						
	Generic event																						
	Increase																						
	Nominal Independent																						
	Partial return																						
	Quantity/Nominal																						
	Rate repricing																						
	Settlement  notice																						
	Substitution																						
	Total return																						
	Undo trade event																						
	User fields																						
Trade issued (EXR)	Counterparty																						
	I/E																						
Trader	Login																						
User definable field	Lists																						
																							
																							
User Access / Security Matrix																							
select usr.M_LABEL as 'USERID', usr.M_DESC AS 'DESC', '' as 'PWID',middle.M_GROUP as 'M_GROUP',																							
case when grp.M_BO_FO = 0 then 'FRONT OFFICE' 																							
                 when grp.M_BO_FO = 1 then 'MIDDLE OFFICE'																							
                when grp.M_BO_FO = 2 then 'BACK OFFICE'      																							
                 else '' end as 'FO/MO/BO',																							
'' as 'PROFILE USER'																							
from TRN_USRD_DBF usr,TRN_USRG_DBF middle,TRN_GRPD_DBF grp 																							
where usr.M_LABEL *= middle.M_USER and middle.M_GROUP *= grp.M_LABEL									
## EOD
目前，世界上大约有30多个主要的外汇市场，它们遍布于世界各大洲的不同国家和地区。根据传统的地域划分，可分为亚洲、欧洲、北美等三大部分，其中，最重要的有伦敦、纽约、东京、新加坡、法兰克福、苏黎士、香港、巴黎、洛杉矶、悉尼等。																																																																																	
在中国的外汇交易者拥有别的时区不能比拟的时间优势，就是能够抓住15点到24点的这个波动最大的时间段，其对于一般的投资者而言都是从事非外汇专业的工作，下午5点下班到24点这段时间是自由时间，正好可以用来做外汇投资，不必为工作的事情分心。一般周末全球都是休市的。周一凌晨5点左右开市。 																																																																																	
																																																																																	
夏令时(北半球4-9月实施夏令时；南半球10-3月实施夏令时)										冬令时																																																																							
地 区 		城 市 		开市时间(GMT) 		收市时间(GMT)				地 区 		城 市 		开市时间(GMT) 		收市时间(GMT)																																																																	
大洋州		惠灵顿		21:00		5:00				大洋州		惠灵顿		20:00		4:00																																																																	
亚 洲		悉尼		23:00		7:00				亚 洲		悉尼		22:00		6:00																																																																	
		东京		0:00		6:30						东京		0:00		6:30																																																																	
		新加坡/香港		1:00		8:00						新加坡/香港		1:00		8:00																																																																	
欧 洲		法兰克福		7:30		16:30				欧 洲		法兰克福		7:30		16:30																																																																	
		伦敦		7:30		15:30						伦敦		8:30		16:30																																																																	
北美洲		纽约		12:30		19:00				北美洲		纽约		13:30		20:00																																																																	
																																																																																	
																																																																																	
Murex is a Trade Book system.是一个用来book金融产品交易的平台，在SCB主要用MUREX book五大类金融衍生产品的交易，CURR（currency also know as FX）; COM(commodity);CRD(Credit);IRD(Interest Rate);SCF(Simple Cash Flow) 流入Murex为USER做市场交易，生成Report。																																																																																	
User在Murex中book了一单trade之后，可以通过MUREX查看这单trade 的 P&L，Cash flow,splictation的值，User可以根据这些值去求这单trade的一阶导数，二阶导数，Murex可以根据导数的变化做风险控制（具体在做RIST 和VAR时会用到）。User可以判断这单trade是否能赚钱，在将来的某一时刻是否能做它。																																																																																	
Saber和 Sophis 也是Trade book system. Sophis在国际上比较知名，是做股票的。																																																																																	
																																																																																	
北半球4-9月实施夏令时；南半球10-3月实施夏令时。																																																																																	
		Actual GMT	The date before Yesterday(Actual Date for people)														Yesterday(Actual Date for peple) - Wensday																									Today(Actual Date for people) - Thursday																								Tomorrow (Actual Date for peple) - Friday														Actual GMT TIME	
	夏令时(GMT)	(大洋洲)新西兰惠灵顿	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16	17	18	19	20	21	22	23	24	(大洋洲)新西兰惠灵顿	夏令时(GMT)
		(亚洲)香港/新加坡	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	13	14	15	16	17	18	19	20	21	22	23	24	(亚洲)香港/新加坡	
		(欧洲)伦敦	20	21	22	23	0	1	2	3	4	5	6	7	7:30	9	10	11	12	13	14	15	15:30	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	7:30	9	10	11	12	13	14	15	15:30	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	7:30	9	10	11	12	13	14	15	15:30	17	18	19	20	21	22	23	24	(欧洲)伦敦	
		(北美洲)纽约	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12	12:30	14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12:30	13	14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	11	12:30	13	14	15	16	17	18	19	20	21	22	23	24	(北美洲)纽约	
		Murex(Live Env) Time	20	0	1	2	3	4	5	6	7	8	9	Yesterday(Murex Date) - Wensday         13				14	15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	10	        Today(Murex Date) - Thursday         14				15	16	17	18	19	20	21	22	23	0	1	2	3	4	5	6	7	8	9	 Tomorrow (Murex Date) - Friday            13				14	15	16	17	18	19	20	21	22	23	0	1	2	3	Murex Date Time	
																																																																																	
																												如果roll date 在周四1AM 才结束(如右图)， SQL要参考如下				0	1	2					Today(Murex Date) - Thursday																																										
																											例： 	insert into MOP_ALL_TDY_REP( M_NB, M_DEST_NB, M_DATE_CMP, M_DATE, M_USR_NAME,M_USR_GROUP, M_ACT_NB1, M_ACT_NB2, M_C_ACT_NB2, FINANCIAL, NON_FIN)																																																					
																												select PAR.M_ACT_NB0 as 'M_NB', 0 as 'M_DEST_NB', PAR.M_DATE_CMP as 'M_DATE_CMP', PAR.M_DATE as 'M_DATE',PAR.M_USR_NAME as 'M_USR_NAME', PAR.M_USR_GROUP as 'M_USR_GROUP',PAR.M_ACT_NB1 as 'M_ACT_NB1', PAR.M_ACT_NB2 as 'M_ACT_NB2', 0 as 'M_C_ACT_NB2', '', ''																																																					
																												from DM_TRN_AUD_REP PAR																																																					
																												WHERE PAR.M_ACTION = 'RPL_DEL' 																																																					
																												--and PAR.M_DATE = @dateValue																																																					
																												and ((PAR.M_DATE = @dateValue) or (PAR.M_DATE_CMP = @dateValue and PAR.M_DATE = @previousDateValue))																																																					
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
	Murex 支持book Murex Date 之前的trade，如何支持mike 没讲. Ctrl+u => END USER查看MUREX Date																																																																																
																																																																																	
	对于SCB来说，（夏令时）每天早上GST 4：00 (8PM GMT)会运行一些JOBs,这些JOBs会做以下事情:																																																																																
	1 roll date，与实际日期同步。在SCB Murex系统中一共有四个Date（FO EOD， BO EOD ，Accounting EOD，Cancel EOD）依次需要被roll, FO EOD ROLL完了，任何人再book trade就是在第二天book trade了，BO EOD 需要被roll是为reporting 用的，mike没讲为什么需要roll一个BO EOD. Accounting EOD是为银行用的，具体mike 没讲， Cancel EOD MIKE 没讲。Murex Date也有节假日，MUREX ROLL DATE 会参考CALENDA，比如周五roll date ， 加一个business Date后会自动跳到下周一。不同的trade适用于不同的Calendar.																																																																																
	1.1	Trade的交易时间，即哪一天开始，到哪一天到期是参考MUERX Date 的，与系统时间无关。																																																																															
	1.2	FO EOD ROLL DATE第一步从上游(如路透社)拿到一些market Data。																																																																															
		Back up deal dump																																																																															
		打印 Transaction Log																																																																															
		FO MARKER    BO MARKER  					WAIT_ACHILLES -> 第一个job																																																																										
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
	2 对上一天的data进行结算，包括market data,trade information 进行copy																																																																																
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																			perl 脚本中根据config参数，定义所有环境的环境变量，如																																																														
																			eod_man.irdfxdev9.cf																																																														
																			eod_env.irdfxdev9.cf																																																														
																			eod_error.irdfxdev9.cf																																																														
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																DMART_GENFDR1																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
										DM_TLM_TRD																																																																							
																																																																																	
																																																																																	
										DM_EXT_AAG																																																																							
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	
																																																																																	

