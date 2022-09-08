coreutils=$(echo 'spy:;@echo $(all_programs) $(noinst_PROGRAMS)' |
            (cd src; make -f Makefile -f - spy | tr -s '\n ' '  '))
mkdir -p src/vg
pwd=`pwd`
srcdir=$pwd/src
_path='export PATH='$srcdir':${PATH#*:}'
pre='#!/bin/sh\n'"$_path"'\n'
n=15 # stack trace depth
log_fd=3 # One can redirect this to file like 3>vg.log
test -e /tmp/cu-vg && suppressions='--supressions=/tmp/cu-vg'
vg="exec /usr/bin/valgrind $suppressions --log-fd=$log_fd \
--leak-check=yes --track-fds=yes --leak-check=full --num-callers=$n"
cat <<EOF > src/vg/gen
for i in $coreutils; do
  printf "$pre$vg -- \$i"' "\$@"\n' > \$i
  chmod a+x \$i
done
EOF
cd src/vg
. ./gen
