---
layout: post
title: 'Shell Script: 番茄式時間管理法'
date: 2013-09-13 13:20
comments: true
categories: [Shell Script, Pomodoro Technique]
---
這是從The clean coder中文版看到的時間方式，本來想要找看看Ubuntun上面有沒有現成軟體可以用。後來看到[有人](http://navaneeth.github.io/blog/2010/12/19/simple-egg-timer-on-linux-for-pomodoro-technique/)使用Shell Script去弄一個。手癢也來做一個自己的版本。

## 番茄式時間管理法簡介
- Francesco Cirillo 在1980年代末提出來的時間管理方式

### 運作原則
1. 設定計時器二十五分鐘，這段時間內專心做事
2. 二十五分鐘時間到休息五分鐘
3. 1 和 2 總共重覆四次後休息長一點的時間


## 測試環境
- Ubuntu 12.04
- bash

## 程式碼

```bash
#!/bin/bash
## Inspired by http://navaneeth.github.io/blog/2010/12/19/simple-egg-timer-on-linux-for-pomodoro-technique/
## info: http://en.wikipedia.org/wiki/Pomodoro_Technique

## Function
function start_timer_minutes() ## Unit is minute
{
    time_elapse_minutes=0
    expire_period_minutes=$1

    while [ $time_elapse_minutes -lt $expire_period_minutes ] ; do
       sleep 60
       time_elapse_minutes=$(($time_elapse_minutes + 1))
       echo "$time_elapse_minutes minute(s) elapse of $expire_period_minutes minutes at iteration $((pomodori+1)).";
    done
}

## Check options
if [ $## -eq 2 ] ; then
    work_period_minutes=$1
    break_period_minutes=$2
elif [ $## -eq 0 ] ; then
    work_period_minutes=25
    break_period_minutes=5
else
    echo $0 expire_period_minutes break_period_minutes
    exit
fi

## Info variables
alert_title="Pomodoro"
start_message="Pomodoro started. You have $work_period_minutes minutes left"
end_message="Pomodoro ended. Take a short break for $break_period_minutes minutes."
max_pomodori=4

## pomodori, 4 iterations
pomodori=0
while [ $pomodori -lt $max_pomodori ] ; do
    ## Work time
    notify-send -u critical -i appointment -t 600 "$alert_title" "$start_message at iteration $((pomodori+1))"
    start_timer_minutes "$work_period_minutes"

    ## Rest time
    if [ $pomodori -lt $((max_pomodori - 1)) ] ; then
		notify-send -u critical -i appointment -t 600 "$alert_title" "$end_message"
		start_timer_minutes "$break_period_minutes"
    fi

    pomodori=$((pomodori + 1))
done

notify-send -u critical -i appointment -t 600 "$alert_title" "Time's up.. Take a longer rest."
echo "Time's up.. Take a longer rest."

```

## 參考資料
- [Wiki: Pomodoro Technique](http://en.wikipedia.org/wiki/Pomodoro_Technique)
- [Simple Egg Timer on Linux for Pomodoro Technique](http://navaneeth.github.io/blog/2010/12/19/simple-egg-timer-on-linux-for-pomodoro-technique/)
