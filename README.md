Run below command as root.

adduser --disabled-password  --quiet --system --home /usr/local/switch --gecos "FreeSWITCH Voice Platform" --ingroup daemon freeswitch
chown -R freeswitch:daemon /usr/local/switch/
chmod -R o-rwx /usr/local/switch/
mkdir /usr/local/switch/run
chown -R freeswitch:daemon /usr/local/switch/run
vi /etc/init.d/freeswitch
### Add below line of code in above run vi editor ###
#!/bin/bash
### BEGIN INIT INFO
# Provides:          freeswitch
# Required-Start:    $local_fs $remote_fs
# Required-Stop:     $local_fs $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Description:       Freeswitch debian init script.
# Author:            Matthew Williams
#
### END INIT INFO
# Do NOT "set -e"
# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/bin
DESC="Freeswitch"
NAME=freeswitch
DAEMON=/usr/local/switch/bin/$NAME
DAEMON_ARGS="-nc"
PIDFILE=/usr/local/switch/var/run/freeswitch/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
FS_USER=freeswitch
FS_GROUP=daemon
# Exit if the package is not installed
[ -x "$DAEMON" ] || exit 0
# Read configuration variable file if it is present
[ -r /etc/default/$NAME ] && . /etc/default/$NAME
# Load the VERBOSE setting and other rcS variables
. /lib/init/vars.sh
# Define LSB log_* functions.
# Depend on lsb-base (>= 3.0-6) to ensure that this file is present.
. /lib/lsb/init-functions
#
# Function that sets ulimit values for the daemon
#
do_setlimits() {
        ulimit -c unlimited
        ulimit -d unlimited
        ulimit -f unlimited
        ulimit -i unlimited
        ulimit -n 999999
        ulimit -q unlimited
        ulimit -u unlimited
        ulimit -v unlimited
        ulimit -x unlimited
        ulimit -s 240
        ulimit -l unlimited
        return 0
}
#
# Function that starts the daemon/service
#
do_start()
{
    # Set user to run as
        if [ $FS_USER ] ; then
      DAEMON_ARGS="`echo $DAEMON_ARGS` -u $FS_USER"
        fi
    # Set group to run as
        if [ $FS_GROUP ] ; then
          DAEMON_ARGS="`echo $DAEMON_ARGS` -g $FS_GROUP"
        fi
        # Return
        #   0 if daemon has been started
        #   1 if daemon was already running
        #   2 if daemon could not be started
        start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --test > /dev/null -- \
                || return 1
        do_setlimits
        start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --background -- \
                || return 2
        # Add code here, if necessary, that waits for the process to be ready
        # to handle requests from services started subsequently which depend
        # on this one.  As a last resort, sleep for some time.
}
#
# Function that stops the daemon/service
#
do_stop()
{
        # Return
        #   0 if daemon has been stopped
        #   1 if daemon was already stopped
        #   2 if daemon could not be stopped
        #   other if a failure occurred
        start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name $NAME
        RETVAL="$?"
        [ "$RETVAL" = 2 ] && return 2
        # Wait for children to finish too if this is a daemon that forks
        # and if the daemon is only ever run from this initscript.
        # If the above conditions are not satisfied then add some other code
        # that waits for the process to drop all resources that could be
        # needed by services started subsequently.  A last resort is to
        # sleep for some time.
        start-stop-daemon --stop --quiet --oknodo --retry=0/30/KILL/5 --exec $DAEMON
        [ "$?" = 2 ] && return 2
        # Many daemons don't delete their pidfiles when they exit.
        rm -f $PIDFILE
        return "$RETVAL"
}
#
# Function that sends a SIGHUP to the daemon/service
#
do_reload() {
        #
        # If the daemon can reload its configuration without
        # restarting (for example, when it is sent a SIGHUP),
        # then implement that here.
        #
        start-stop-daemon --stop --signal 1 --quiet --pidfile $PIDFILE --name $NAME
        return 0
}
case "$1" in
  start)
        [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
        do_start
        case "$?" in
                0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
                2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
  stop)
        [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
        do_stop
        case "$?" in
                0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
                2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
  status)
       status_of_proc -p $PIDFILE $DAEMON $NAME && exit 0 || exit $?
       ;;
  #reload|force-reload)
        #
        # If do_reload() is not implemented then leave this commented out
        # and leave 'force-reload' as an alias for 'restart'.
        #
        #log_daemon_msg "Reloading $DESC" "$NAME"
        #do_reload
        #log_end_msg $?
        #;;
  restart|force-reload)
        #
        # If the "reload" option is implemented then remove the
        # 'force-reload' alias
        #
        log_daemon_msg "Restarting $DESC" "$NAME"
        do_stop
        case "$?" in
          0|1)
                do_start
                case "$?" in
                        0) log_end_msg 0 ;;
                        1) log_end_msg 1 ;; # Old process is still running
                        *) log_end_msg 1 ;; # Failed to start
                esac
                ;;
          *)
                # Failed to stop
                log_end_msg 1
                ;;
        esac
        ;;
  *)
        #echo "Usage: $SCRIPTNAME {start|stop|restart|reload|force-reload}" >&2
        echo "Usage: $SCRIPTNAME {start|stop|restart|force-reload}" >&2
        exit 3
        ;;
esac
exit 0
### Code end here ###

chmod +x /etc/init.d/freeswitch
update-rc.d freeswitch defaults
cd /usr/local/switch/etc/freeswitch/directory/default
vi 1000.xml
chown freeswitch:daemon 1000.xml
vi 1001.xml
chown freeswitch:daemon 1001.xml
vi /usr/local/switch/etc/freeswitch/dialplan/default.xml
<extension name="call-group-order">
      <condition field="destination_number" expression="^83(\d{2})$">
        <action application="bridge" data="{leg_timeout=15,ignore_early_media=true}${group(call:$1@${domain_name}:order)}"/>
      </condition>
</extension>
Use password for 1000 and 1001 as 123456
run reloadxml on freeswitch cli to reload the above mentioned changes
Now If you are dialling from 1000 user then dial 831001 it will connect to 1001 user or perfomr vice-versa.