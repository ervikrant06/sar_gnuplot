## sar_gnuplot
## Using gnuplot for drawing the sar graph
#!/bin/bash
if [ $# -ne 2 ] ; then
echo "Usage : $0 option sar file name"
exit 1
fi
function cpu() {
LANG=C /usr/bin/sar -u -f $2 | /usr/bin/awk '!/^Average|^Linux|CPU|^$/ {print $1,$3,$5,$6, $3+$5+$6}' > /root/test/$filename.txt
cat > /root/sar.plot<<EOF
set terminal x11 title "CPU graph"
set title "CPU util"
set xdata time
set timefmt '%H:%M:%S'
set xlabel 'Time'
set ylabel 'CPU utilization'
set format x '%H:%M:%S'
set yrange [0:]
plot '/root/test/$filename.txt' using 1:2 title 'user util' with lines, \
     '/root/test/$filename.txt' using 1:3 title 'system util' with lines, \
     '/root/test/$filename.txt' using 1:4 title 'iowait util' with lines, \
     '/root/test/$filename.txt' using 1:5 title 'total util' with lines
EOF
echo "Plotting"
/usr/bin/gnuplot -persist /root/sar.plot 2> /dev/null
rm -rf /root/test/$filename.txt
}
function loadavg() {
LANG=C /usr/bin/sar -q -f $2 | /usr/bin/awk '!/^Average|^Linux|ldavg|^$/ {print $1,$4,$5,$6}' > /root/test/$filename.txt
cat > /root/sar.plot<<EOF
set terminal x11 title "loadavg graph"
set title "load avg"
set xdata time
set timefmt '%H:%M:%S'
set xlabel 'Time'
set ylabel 'loadavg utilization'
set format x '%H:%M:%S'
set yrange [0:]
plot '/root/test/$filename.txt' using 1:2 title '1 min' with lines, \
     '/root/test/$filename.txt' using 1:3 title '5 min' with lines, \
     '/root/test/$filename.txt' using 1:4 title '15 min' with lines
EOF
echo "Plotting"
/usr/bin/gnuplot -persist /root/sar.plot 2> /dev/null
#rm -rf /root/test/$filename.txt
}


filename=`file $2 | awk -F: '{print $1}' | awk -F/ '{print $NF}'`
while getopts :u:q options
do
case $options in
u) cpu;;
q) loadavg;;
*) "Invalid arg" ;;
esac
done
