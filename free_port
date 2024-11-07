#!/usr/bin/env bash

set -a
#set -x

if ! [[ $1 =~ ^[0-9]+$ ]]; then 
echo "Command expects the desired port number as a single argument, aborting..." >&2
echo "(invoked as '$0')" >&2
exit 1
fi

if [[ $1 -lt 1 || $1 -gt 65535 ]]; then
echo "You passed a port number of $1 which is outside the range 1-65535, aborting..." >&2
exit 2
fi

# A bash-array that contains a list of all the ports currently occupied on the host
PORTS=( 0 $(netstat -tuln | grep -Ei '^(tcp|udp).*LISTEN *$' | awk '{print $4}' | \
    awk -F ':+' '{print $2}' | sort -gu) 65536 )

if [[ ${#PORTS[*]} -le 2 ]]; then
echo "Did you forget to pass '--network=host' to the docker run command?" >&2
exit 3
fi

# The desired port number is fed as an argument
DESIRED_PORT=$1

# Binary-search state variables, and a counter to put a hard limit on the number of
# iterations and for debugging
START=0
END=${#PORTS[*]}
COUNTER=0

for (( CURSOR = ${#PORTS[*]} / 2 ; DELTA = ${PORTS[$CURSOR]} - $DESIRED_PORT ; COUNTER = COUNTER + 1 )) ; do 
# printf "counter = %d\n" $COUNTER
if [[ $COUNTER -gt 99 ]]; then break; fi 

# printf "%d: ports[%d] = %d (delta=%d)\n" $COUNTER $CURSOR ${PORTS[$CURSOR]} $DELTA
if [[ DELTA -lt 0 ]]; then (( START = $CURSOR + 1 )); else (( END = $CURSOR )); fi
(( CURSOR = $START + ( $END - $START ) / 2 ))

# printf "%d: search range is [%d,%d), cursor = %d \n" $COUNTER $START $END $CURSOR
if [[ $START -eq $END || ( ${PORTS[$CURSOR]} -lt $DESIRED_PORT && \
    ${PORTS[(( $CURSOR + 1 ))]} -gt $DESIRED_PORT ) ]]; then break; fi
done

# printf "%d: ports[%d] = %d\n" $COUNTER $CURSOR ${PORTS[$CURSOR]}

RESULT=$DESIRED_PORT
COUNTER=0
until [[ $RESULT -gt ${PORTS[$CURSOR]} && $RESULT -lt ${PORTS[(( $CURSOR + 1 ))]} ]]; do
# printf "%d: candidate result=%d, ports={%d,%d,...}\n" $COUNTER $RESULT ${PORTS[$CURSOR]} ${PORTS[(( $CURSOR + 1 ))]}

(( RESULT = $RESULT + 1 ))
if [[ $RESULT -gt 65535 ]]; then
echo 0
exit 4
fi

if [[ $RESULT -ge ${PORTS[(( $CURSOR + 1 ))]} ]]; then (( CURSOR = $CURSOR + 1 )); fi

(( COUNTER = $COUNTER + 1 ))
if [[ $COUNTER -gt 99 ]]; then break; fi 
done;

# printf "PORTS[%d..%d]={ %d }\n" 0 ${#PORTS[*]} ${PORTS[*]}
# printf "%d: result=%d, cursor=%d, ports={%d,%d,...}\n" $COUNTER $RESULT $CURSOR ${PORTS[$CURSOR]} ${PORTS[(( $CURSOR + 1 ))]}

echo $RESULT