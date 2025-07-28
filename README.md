#!/system/bin/sh
MODDIR=${0%/*}
count=0
modulesParent=/data/adb/modules
CheckDeepRecovery=${MODDIR}/.DeepRec

while [ $count -lt 3 ]; do
	if [ "$(getprop sys.boot_completed)" == "1" ]; then
	     mv -f ${MODDIR}/AutoRec* /dev/null 2>/dev/null
		 mv -f ${MODDIR}/.DeepRec /dev/null 2>/dev/null
		 echo -e "开机成功，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")\n" >>${MODDIR}/Log.txt
		 break
	fi
	event=$(timeout 1 getevent -qlc 1 | grep -i "KEY_VOLUME" | grep -iw "DOWN")
	if [ -n "$event" ]; then
		echo "音量键第$((${count}+1))次按下，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")" >>${MODDIR}/Log.txt
		count=$((count + 1))
	fi
done

if [[ "$count" -eq "3" ]]; then
	if [ -f $CheckDeepRecovery ]; then
		mv -f $CheckDeepRecovery /dev/null 2>/dev/null
		mv -f /data/system/users/0/package-restrictions* /dev/null 2>/dev/null
		echo "尝试还原系统应用，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")" >>${MODDIR}/Log.txt
		settings delete secure enabled_accessibility_services
		echo "尝试禁用无障碍功能，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")" >>${MODDIR}/Log.txt
		mv -f /data/property/persistent_properties /data/property/bak_persistent_properties
		echo "尝试恢复系统prop属性，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")" >>${MODDIR}/Log.txt
		mv -f /data/system/package_cache /data/system/package_cache_bak 2>/dev/null
		echo "尝试重置系统缓存，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")" >>${MODDIR}/Log.txt
	
	else
		for path in $(ls $modulesParent); do
	       	if [ "$path" != "RescueBrick_TimeMod" ]; then
		        echo "尝试禁用 ${path} 模块，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")" >>${MODDIR}/Log.txt
		        touch "$modulesParent/$path/disable"
		    fi
	    done
		touch "$CheckDeepRecovery"
	fi
	
	echo -e "开机中断，发生时间：$(date "+%Y年%m月%d日%H:%M:%S")\n" >>${MODDIR}/Log.txt
	reboot
fi


