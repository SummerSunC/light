```
#!/bin/bash
temp=$1
if [ -z $temp ]
then
  temp=5
fi
let temp=$temp+1
echo "=================start============"
echo "fileCount | blockCount | TotalSize |upFiles"
for((i=1;i<=$temp;i++));do
   info=`/opt/tfs/bin/ssm -s 10.129.195.71:8100 -i block|tail -2|head -1|awk '{print $3 ,$2 , $4}'`;
# echo ${info} |awk '{print $1 "\t" "\t" $2 "\t" "\t" $3}'
  if [ ${i} -eq 1 ];
  then
    fristFiles=`echo ${info}|awk '{print $1}'`;
  elif [ ${i} -eq $temp ];
  then
    let nFiles=`echo ${info}|awk '{print $1}'`
    let tFiles=${nFiles}-${pFiles}
    echo ${info} ${tFiles}|awk '{print $1 "\t" "\t" $2 "\t" "\t" $3 "\t" "\t" $4"/sec"}'
    echo "#########################################################"
    let lastFiles=`echo ${info}|awk '{print $1}'`
    let totalFiles=${lastFiles}-${fristFiles}
    let sec=${temp}-1
    let aveFile=${totalFiles}/${sec}
    echo "# Transport [${totalFiles}] Files in [${sec}] seconds,Average : [${aveFile}] pre second #"
    echo "#########################################################"
  else
    let nFiles=`echo ${info}|awk '{print $1}'`
    let tFiles=${nFiles}-${pFiles}
    echo ${info} ${tFiles}|awk '{print $1 "\t" "\t" $2 "\t" "\t" $3 "\t" "\t" $4"/sec"}'
  fi
  echo "---------------------------------------------------------------"
  let pFiles=`echo ${info}|awk '{print $1}'`
done;
echo "================end==============="
```