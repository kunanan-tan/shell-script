# shell-script
#1. validate_file.sh

#get parameter/path config.sh
source /config.sh

cd $FILE_PATH

year_valid=`date +%Y`

mon_valid=`date +%m`

day_valid=`date +%d`

TODAY_VALID=${year_valid}-${mon_valid}-${day_valid}

pipeCount=

lengthCount=

#loop find .CSV
for file in *.CSV; do

file_no_ext=${file%.*}

#change file name / แก้ไขชื่อไฟล์
mv $file "$file_no_ext"_ORG.CSV

#condition check file name / เช็คเงื่อนไขจากชื่อที่ได้จาก loop
if [ "$file_no_ext" == "test1" ]; then
    pipeCount=12
    lengthCount=100
elif [ "$file_no_ext" == "test2" ]; then
    pipeCount=12
    lengthCount=200
elif [ "$file_no_ext" == "test3" ]; then
    pipeCount=3
    lengthCount=300
elif [ "$file_no_ext" == "test4" ]; then
    pipeCount=57
    lengthCount=400
elif [ "$file_no_ext" == "test5" ]; then
    pipeCount=2
    lengthCount=500
fi

#check empty / เช็คว่าต้องมีข้อมูล 
if [ ! -z "$pipeCount" ] && [ ! -z "$lengthCount" ]; then
   
        #check total record / เช็คจำนวนแถวทั้งหมด
        totalORGCount=$(awk 'END{print NR}' "$file_no_ext"_ORG.CSV)

        #find pipe(|) pipecount , without last row and find total row / หา pipe ที่ไม่่ตรงกับ pipecount , ไม่เอาแถวสุดท้าย และนับจำนวนทั้งหมด
        count1=$(awk -F"|" 'NF-1 != '$pipeCount' && NR != '$totalORGCount' {print $0}' "$file_no_ext"_ORG.CSV | wc -l)
        #without last row totalORGCount and loop check length lengthCount / ไม่เอาแถวสุดท้ายและ loop เช็คแต่ละฟิล lengthCount และนับจำนวนแถวนั้น
        count2=$(awk -F"|" 'NR != '$totalORGCount' { for(i=1; i<=NF; i++) { if (length($i) > '$lengthCount') { print $0; break; } } }' "$file_no_ext"_ORG.CSV | wc -l)
        #บวกกัน
        total=$(expr $count1 + $count2)

        #gen new file and check lengthCount / แถวนั้นต้องตรงกัน pipeCount และไม่เอาบรรทัดสุดท้าย โดย loop เช็ค lengthCount ให้ไม่ต้อง gen ใส่ไฟล์ใหม่
        awk -F"|" 'NF-1 == '$pipeCount' && NR != '$totalORGCount' {
            for(i=1; i<=NF; i++) {
                if (length($i) > '$lengthCount') {
                    next;
                }
            }
            print $0;
        }' "$file_no_ext"_ORG.CSV > "$file_no_ext"_$TODAY.CSV

        totalCount=$(wc -l "$file_no_ext"_$TODAY.CSV | awk '{print $1}')

        echo "T|$TODAY_VALID|$totalCount|"$file_no_ext"_$TODAY.CSV" >> "$file_no_ext"_$TODAY.CSV

        #check total greter than 0
        if [ $total -gt 0 ]; then

            #gen log err
            echo ">>$file_no_ext"_ORG.CSV >> $logvalidatefile
            awk -F"|" 'NF-1 != '$pipeCount' && NR != '$totalORGCount' {print $0}' "$file_no_ext"_ORG.CSV >> $logvalidatefile
            awk -F"|" 'NR != '$pipeCount' { for(i=1; i<=NF; i++) { if (length($i) > '$lengthCount') { print $0; break; } } }' "$file_no_ext"_ORG.CSV >> $logvalidatefile

        fi
fi
done
