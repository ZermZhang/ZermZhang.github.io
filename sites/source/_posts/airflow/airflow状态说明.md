---
title: airflow状态说明
tags: ['airflow','调度器','状态']
date: 2022-04-23
categories: ['airflow']
---
在airflow中，可以通过对pipeline中的不同task赋予不同的状态(state)说明当前任务的执行进度。通过airflow的状态机制，可以对当前的任务执行进度和状态进行很好的把控，及时了解指定任务的情况。

其中，airflow在更新到2.0版本后，相较于之前的1.9.0版本，airflow对任务状态进行了进一步的细化说明。本文主要是为了对airflow的基础状态信息进行一个简单的介绍和记录。
<!--more-->

# 1. airflow状态说明
![airflow-all-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-all-state.2g0opkilfqck.webp)

## 1.1 通过颜色区分
airflow中的状态标识蕾丝红绿灯的状态区分，主要分成了红、黄、绿三种基础状态。在此基础上，针对三种不同的颜色范围，进行了进一步的细化标识来表明不同的细化状态。

## 1.2 通过粒度区分
除此之外，airflow状态还会根据标明级别，分成了dag状态和task状态两大类，不同类别的状态主要有如下几个小类：
1. Dagrun：SUCCESS, RUNNING, FAILED
2. Task：SUCCESS, RUNNING, FAILED, UPSTREAM_FAILED, SKIPPED, UP_FOR_RETRY, UP_FOR_RESCHEDULE, QUEUED, NONE, SCHEDULED

其中，Task状态主要是针对Dagrun状态的细化说明，因为Dag中不同task因为执行时间、顺序和依赖的不同，会有不同的状态表现。

## 1.3 通过状态范围区分
同时，可以通过是否会继续进行状态变更，分成完成（finished）和未完成（unfinished）两大类。
* 完成状态代表airflow scheduler不会对当前task/dagrun进行监控。
* 未完成状态代表当前task/dagrun的状态还会发生变更，最终会达到一个完成状态。


# 2. airflow状态的详细介绍

### 2.1 SUCCESS（green｜dagrun｜task｜finished）
![airflow-success-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-success-state.2n720i3oq4u0.webp)
* success状态说明当前任务在执行过程中没有检测到错误信息，并正常完成。
* airflow scheduler上报状态后，释放针对当前任务的监控。
* excutor执行完成对应任务
* 颜色：墨绿色

### 2.2 RUNNING（green｜dagrun｜task｜unfinished）
![airflow-running-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-running-state.2hjc0rbgou60.webp)
* running状态代表当前的任务执行执行中
* airflow scheduler将该任务提交到了executor上，并对当前任务通过heartbeat进行监控
* executor正在对对应的实际任务进行执行
* 颜色：浅绿色
* 后续状态：SUCCESS、RETRY、FAILED

### 2.3 FAILED（red｜dagrun｜task｜finished）
![airflow-failed-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-failed-state.6vbtjqjxw8g0.webp)
* failed状态说明当前任务执行失败，返回了错误状态码，airflow监控到了当前错误，表现在web上
* airflow scheduler会上报当前任务failed壮体啊，和下游任务的UPSTREAM_FAILED状态
* executor执行完成对应任务
* 颜色：红色

### 2.4 UPSTREAM_FAILED（red｜task｜finished）
![airflow-upstream_upfailed-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-upstream-failed-state.3m701qkvfc80.webp)
* upstream_failed状态说明当前任务的上游任务发生错误，上有task处于FAILED状态，当前任务并没有开始执行
* airflow scheduler会直接上报当前任务状态，不会再将当前任务进行调度
* executor没有执行对应的任务
* 颜色：棕黄色

### 2.5 SKIPPED（red｜task｜finished）
![airflow-skipped-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-skipped-state.2oekru2jpa60.webp)
* skipped状态代表当前任务被跳过，没有进行执行，经常发生在branchOperator未触发的分支上
* airflow scheduler绕过了当前任务，并没有调度
* executor没有执行对应的任务
* 颜色：浅红色

### 2.6 UP_FOR_RETRY（yellow｜task｜unfinished）
![airflow-up_for_retry-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-up-for-retry-state.h2tpvfwe4lc.webp)
* up_for_retry代表当前任务执行失败并在等待重试中
* airflow scheduler会在retry interval之后，下一次heartbeat到达时重新调度该任务
* executor上次执行对应任务失败，上次执行过程已经结束，当前没有执行对应任务
* 颜色：黄色

### 2.7 UP_FOR_RESCHEDULE（green｜task｜unfinished）
![airflow-up_for_reschedule-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-up_for_reschedule-state.1bo58g36z6qo.webp)
* up_for_reschedule是在airflow1.10.2中引入的心状态，常用在Sensor中，可以防止过度消耗slots
* airflow scheduler会对当前的任务进行定时尝试，防止因为长时间处在调度过程中而占据worker slots，从而导致worker锁死，无法执行其他任务
* executor定时执行对应任务
* 颜色：浅绿色

### 2.8 QUEUED（grey｜task｜unfinished）
![airflow-queued-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-queued-state.13weqa9jfna8.webp)
* queued状态代表当前任务已经被调度，但是正在等待一个可用的executor slots
* airflow scheduler已经将该任务调度，正在排队中，当前的pool中没用slots为0
* executor没有空闲的slots执行该任务，或者提交到指定的executor的concurrency已经最大，无法在接受新任务
* 颜色：灰色

### 2.9 NO_STATUS（no｜task｜unfinished）
![airflow-no_status-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-no_status-state.3b4t9j1ta8k0.webp)
* no_status代表当前任务还没有被调度到，前面任务正在执行
* 颜色：无颜色

### 2.10 SCHEDULED（yellow｜task｜unfinished）
![airflow-scheduled-state](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/airflow-scheduled-state.7io0ibe2l5s0.webp)
* scheduled状态代表该任务已经被触发
* airflow scheduler调度该任务，但是并没有open slots（=all_slots - running_slots - queued_slots），正在轮询状态中
* excutor没有空闲的slots执行该任务
* 颜色：棕色
