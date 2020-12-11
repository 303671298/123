# ! /usr/bin/env python
# -*- coding=utf-8 -*-
import random
import sys
import numpy as np
import pandas as pd

import matplotlib.pyplot as plt
import matplotlib.patches as patches
import matplotlib.gridspec as gridspec
from matplotlib.ticker import MultipleLocator

from datetime import datetime
from FileBase import ConfigFile,FileBase
from DatetimeBase import DatetimeBase
from LogBase import LogBase

from lib.Common import Common
from lib import G
import collections
import math
import time
import decimal
datetimebase = DatetimeBase()


plt.switch_backend('agg')
#用来正常显示中文标签
#plt.rcParams['font.sans-serif'] = ['Microsoft YaHei']
plt.rcParams['font.sans-serif']=['SimHei']
plt.rcParams['axes.unicode_minus']=False


class Reportor():
    def __init__(self):
        self._logbase = LogBase()
        self._filebase = FileBase()
        self._configfile = ConfigFile()
        self.__config = self._configfile.getAllConfigFile()
        self._logbase.info('config : {}'.format(self.__config), func=sys._getframe().f_code.co_name)
        self._common = Common()
        self._focus_project_list = ['huting','浑天']
        self._focus_color_mapping = {'huting':'lightgreen', '浑天':'lightgreen','other':'lightsteelblue'}

    def report(self):
        self._logbase.info('begin...', func=sys._getframe().f_code.co_name)
        data_date = G.input_parms['schedate']
        date_before_7day = str(datetimebase.dateAdd(dt=data_date, days=-7)).split(' ')[0]
        date_before_5week = str(datetimebase.dateAdd(dt=data_date, days=-35)).split(' ')[0]
        date_before_30day = str(datetimebase.dateAdd(dt=data_date, days=-30)).split(' ')[0]
        # 创建figure窗口
        fig1 = plt.figure(num=1, figsize=(20, 80), facecolor='whitesmoke')
        gs = gridspec.GridSpec(14, 2, height_ratios=[1, 1.5, 1.5, 5, 6, 6, 7, 0.3, 7, 0.2, 7, 0.3, 7,7])

        # 子图1：标题
        self._logbase.info('genaration figure title...', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[0, :], facecolor='whitesmoke')
        self.figure_title(ax,data_date)
        self._logbase.info('genaration figure title...done', func=sys._getframe().f_code.co_name)

        # 子图2：当前数据
        self._logbase.info('genaration overview current...', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[1, :], facecolor='whitesmoke')
        self.overview_current(ax,data_date)
        self._logbase.info('genaration overview current....done', func=sys._getframe().f_code.co_name)

        # 子图3：近七天数据
        self._logbase.info('genaration overview last7day...', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[2, :], facecolor='whitesmoke')
        self.overview_last7day(ax,data_date,date_before_7day,date_before_5week)
        self._logbase.info('genaration overview last7day....done', func=sys._getframe().f_code.co_name)

        # 子图4-1：项目任务数
        self._logbase.info('genaration dataflow count....', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[3, 0], facecolor='w')
        self.dataflow_count(ax,data_date)
        self._logbase.info('genaration dataflow count....done', func=sys._getframe().f_code.co_name)

        # 子图4-2：项目耗时
        self._logbase.info('genaration project elapse time....', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[3, 1], facecolor='w')
        self.project_elapse_time(ax,data_date,date_before_7day)
        self._logbase.info('genaration project elapse time....done', func=sys._getframe().f_code.co_name)

        # 子图5：项目耗时的分段图
        self._logbase.info('genaration project elapse time dis....', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[4, 0], facecolor='w')
        self.project_elapse_time_dis(ax,data_date,date_before_7day)
        self._logbase.info('genaration project elapse time dis....done', func=sys._getframe().f_code.co_name)

        # 子图5-1：各项目每天执行实际耗时
        self._logbase.info('genaration project elapse time dis....everyday', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[4, 1], facecolor='w')
        self.project_elapse_time_everyday(ax, data_date, date_before_7day)
        self._logbase.info('genaration project elapse time dis....done everyday', func=sys._getframe().f_code.co_name)

        # 子图6：top10
        self._logbase.info('genaration dataflow top....', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[5, 0], facecolor='whitesmoke')
        self.top_title(ax)
        ax = plt.subplot(gs[5, 1], facecolor='w')
        self.dataflow_top(ax,data_date,date_before_7day)
        self._logbase.info('genaration dataflow top....done', func=sys._getframe().f_code.co_name)


        # 子图7：甘特图
        #for i in range(7):
        self._logbase.info('genaration last 7 day scheduler...', func=sys._getframe().f_code.co_name)
        dd = str(datetimebase.dateAdd(dt=data_date, days=-1)).split(' ')[0]
        ax = plt.subplot(gs[6, :], facecolor='w')
        self.scheduler_gantt(ax, dd)
        self._logbase.info('genaration last 7 day scheduler...done', func=sys._getframe().f_code.co_name)


        # 子图8：完成时效图
        self._logbase.info('genaration process rate...', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[8, :], facecolor='w')
        self.process_rate(ax)
        self._logbase.info('genaration process rate...done', func=sys._getframe().f_code.co_name)

        # 子图9：各项目近30天完成时点图
        self._logbase.info('month genaration process rate...', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[10, :], facecolor='w')
        self.process_rate_month(ax,data_date,date_before_30day)
        self._logbase.info('month genaration process rate...done', func=sys._getframe().f_code.co_name)

        # 子图10：浑天里程碑执行点
        self._logbase.info(' huting milepost...', func=sys._getframe().f_code.co_name)
        ax = plt.subplot(gs[12, :], facecolor='w')
        self.milepost_week(ax, data_date, date_before_7day)
        self._logbase.info('huting milepost...done', func=sys._getframe().f_code.co_name)

        # plt.show()
        file = self._filebase.filePathJoin(self.__config['player']['report_home_path'], 'HisenReportWeekly-{}-{}.png'.format(data_date, random.randint(1000, 9999)))
        self._logbase.info('save file:{}'.format(file), func=sys._getframe().f_code.co_name)
        plt.subplots_adjust(bottom=0.01, top=1, left=0.015, right=0.995, hspace=0.1, wspace=0.1)


        #plt.subplots_adjust(top=1, bottom=0, right=1, left=0, hspace=0.1, wspace=0.1)
        plt.margins(0, 0)
        #fig1.show()

        fig1.savefig(file, dpi=100, pad_inches = 0, facecolor='whitesmoke')
        self._logbase.info('all done', func=sys._getframe().f_code.co_name)


        return file

    def figure_title(self,ax,data_date):
        # 设置标题-------------------------------------------
        plt.xlim((0, 100))
        plt.ylim((0, 50))

        plt.text(40, 30, u'大数据平台调度周报', fontsize=20)
        plt.text(80, 15, u'{} by 大数据平台组'.format(data_date), fontsize=10, color='grey')

        # 去掉边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        ax.spines['bottom'].set_visible(False)
        # 去掉刻度
        ax.set_xticks([])
        ax.set_yticks([])

    def overview_current(self,ax,data_date):
        plt.xlim((0, 222))
        plt.ylim((0, 22))

        # 接入项目数
        sql = '''select count(distinct project_group) as cnt from project_ext'''
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)
        ax.add_patch(patches.Rectangle((2, 2), 40, 18, facecolor="#9F79EE"))
        plt.text(22, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(22, 6, u'接入项目数', fontsize=11, color='white', horizontalalignment='center')

        # DAG总数
        sql = '''select count(1) from dataflow2 d where enable = 1'''
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)
        ax.add_patch(patches.Rectangle((46, 2), 40, 18, facecolor="#EE9572"))
        plt.text(66, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(66, 6, u'Airflow DAG总数', fontsize=11, color='white', horizontalalignment='center')

        # Task数量
        sql = 'select count(1) from dataflow2 where enable = 1'
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        data = self._common.mysql_select(sql, withcolumns=False)
        ax.add_patch(patches.Rectangle((91, 2), 40, 18, facecolor="#FA8072"))
        plt.text(111, 11, data[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(111, 6, u'Task数量', fontsize=11, color='white', horizontalalignment='center')

        # 当前dataflow数
        sql = '''select count(1) from run2 where date(created_time) = '{data_date}' and status = 'Success' '''
        sql = sql.format(data_date=data_date)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)

        result = self._common.mysql_select(sql, withcolumns=False)
        ax.add_patch(patches.Rectangle((136, 2), 40, 18, facecolor="#5CACEE"))
        plt.text(156, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(156, 6, u'当前dataflow数', fontsize=11, color='white', horizontalalignment='center')

        # dataflow2平均耗时
        sql = '''select sum(elapse_time)/count(1) as avg from run2 where date(created_time) = '{data_date}' and status = 'Success' '''
        sql = sql.format(data_date=data_date)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)
        ax.add_patch(patches.Rectangle((180, 2), 40, 18, facecolor="#EE9572"))
        plt.text(200, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(200, 6, u'dataflow2平均耗时(秒)', fontsize=11, color='white', horizontalalignment='center')

        # 去掉边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        ax.spines['bottom'].set_visible(False)

        # 去掉刻度
        ax.set_xticks([])
        ax.set_yticks([])

        plt.title(u'截至当前', fontsize=12, color='grey', loc='left')

    def overview_last7day(self,ax,data_date,date_before_7day,date_before_5week):

        plt.xlim((0, 222))
        plt.ylim((0, 22))

        # 近7天Task出错量
        sql = ''' select count(distinct dataflow) as cnt from run2 where date(created_time) between '{date_before_7day}' and '{data_date}' and status = 'Failed' and dataflow not like 'pase%' and dfid <> 9999 '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)

        ax.add_patch(patches.Rectangle((2, 2), 40, 18, facecolor="#FA8072"))
        plt.text(22, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(22, 6, u'近7天Task出错量', fontsize=11, color='white', horizontalalignment='center')

        # 近7天Task超时量
        sql = ''' select sum(case when elapse_time > 30*60 then 1 else 0 end) as cnt
                    from (
                    select dataflow,sum(elapse_time) elapse_time 
                     from run2 
                    where date(created_time) between '{date_before_7day}' and '{data_date}' and status ='Success' group by dataflow
                    ) a '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)

        sql = ''' select sum(case when elapse_time > 60*60 then 1 else 0 end) as cnt
                    from (
                    select dataflow,sum(elapse_time) elapse_time 
                     from run2 
                    where date(created_time) between '{date_before_5week}' and '{date_before_7day}' and status ='Success' group by dataflow
                    ) a'''
        sql = sql.format(date_before_7day=date_before_7day, date_before_5week=date_before_5week)

        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        s = self._common.mysql_select(sql, withcolumns=False)

        avg_s = float(s[0][0]) / 4
        r = u'{:.0%}'.format((float(result[0][0]) - avg_s) / avg_s)

        if result > avg_s:
            c = 'r'
            ax.text(72, 11, u'↑', fontsize=15, color=c)
        else:
            c = 'green'
            ax.text(72, 11, u'↓', fontsize=15, color=c)

        ax.add_patch(patches.Rectangle((46, 2), 40, 18, facecolor="#9F79EE"))
        ax.text(63, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        ax.text(74, 11, r, fontsize=8, color=c)
        ax.text(66, 6, u'近7天Task超时量(>1H)', fontsize=11, color='white', horizontalalignment='center')

        # 近7天Task完成率(8H)
        sql = '''
        select sum(case when (TIMESTAMPDIFF(HOUR, end_time, begin_time))*24*60 < 8*60 then 1 else 0 end) as sum,count(1) as cnt
                    from (
                    select date(created_time) data_date,dataflow,min(begin_time) begin_time, max(end_time) end_time
                     from run2 where date(created_time) between '{date_before_7day}' and '{data_date}' and status='Success'
                     group by date(created_time),dataflow
                    ) a
        '''

        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)

        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)
        r_7 = round(int(result[0][0]) * 1.00 / result[0][1], 4) * 100

        sql = '''select sum(case when (TIMESTAMPDIFF(HOUR,end_time,begin_time))*24*60 < 8*60 then 1 else 0 end) as sum,count(1) as cnt
                    from (
                    select date(created_time) data_date,dataflow,min(begin_time) begin_time, max(end_time) end_time
                     from run2 where date(created_time) between '{date_before_5week}' and '{date_before_7day}' and status='Success'
                     group by date(created_time),dataflow
                    ) a '''
        sql = sql.format(date_before_7day=date_before_7day, date_before_5week=date_before_5week)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        f =self._common.mysql_select(sql, withcolumns=False)

        r_4w = round(float(f[0][0]) * 1.00 / float(f[0][1]), 4) * 100

        if r_7 > r_4w:
            c = 'green'
            i=0.5
            ax.text(122, 11, u'↑', fontsize=15, color=c)
        else:
            c = 'r'
            i=-0.5
            ax.text(122, 11, u'↓', fontsize=15, color=c)

        ax.add_patch(patches.Rectangle((91, 2), 40, 18, facecolor="#EE9572"))
        ax.text(110, 11, '{}%'.format(r_7), fontsize=25, color='white', horizontalalignment='center')
        ax.text(124, 11, '{}%'.format(int(r_7 - r_4w+i)), fontsize=8, color=c)
        ax.text(111, 6, u'近7天Task准时率(<=8H)', fontsize=11, color='white', horizontalalignment='center')

        # 近7天执行dataflow2总数
        sql = ''' select count(1) as cnt
                    from run2
                   where date(created_time) between '{date_before_7day}' and '{data_date}' and status ='Success' '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        cnt_7 = self._common.mysql_select(sql, withcolumns=False)

        sql = ''' select count(1) as cnt
                    from run2
                   where date(created_time) between '{date_before_5week}' and '{date_before_7day}' and status ='Success' '''
        sql = sql.format(date_before_7day=date_before_7day, date_before_5week=date_before_5week)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        cnt_4w = self._common.mysql_select(sql, withcolumns=False)
        avg = float(cnt_4w[0][0]) / 4

        if float(cnt_7[0][0]) > avg:
            c = 'green'
            ax.text(167, 11, u'↑', fontsize=15, color=c)
        else:
            c = 'r'
            ax.text(167, 11, u'↓', fontsize=15, color=c)

        ax.add_patch(patches.Rectangle((136, 2), 40, 18, facecolor="#9F79EE"))
        ax.text(156, 11, cnt_7[0][0], fontsize=25, color='white', horizontalalignment='center')
        ax.text(169, 11, '{:.0%}'.format((float(cnt_7[0][0]) - avg) / avg), fontsize=8, color=c)
        ax.text(156, 6, u'近7天执行dataflow2总数', fontsize=11, color='white', horizontalalignment='center')

        # 近7天dataflow2出错数
        sql = ''' select count(1) as cnt from run2 where date(created_time) between '{date_before_7day}' and '{data_date}' and status = 'Failed' and dataflow not like 'pase%' and dfid<>9999 '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        result = self._common.mysql_select(sql, withcolumns=False)
        ax.add_patch(patches.Rectangle((180, 2), 40, 18, facecolor="#5CACEE"))
        plt.text(200, 11, result[0][0], fontsize=25, color='white', horizontalalignment='center')
        plt.text(200, 6, u'近7天dataflow2出错总数', fontsize=11, color='white', horizontalalignment='center')

        # 去掉边框

        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        ax.spines['bottom'].set_visible(False)

        # 去掉刻度
        ax.set_xticks([])
        ax.set_yticks([])

        plt.title(u'最近7天', fontsize=12, color='grey', loc='left')

    def dataflow_count(self,ax,data_date):

        plt.xlim((0, 100))
        plt.ylim((0, 100))
        sql = '''select p.project_group,count(1) cnt
        from dataflow2 d
        join run2 r on d.dfid = r.dfid
        join project_ext p on d.project = p.project
        where date(r.created_time) = '{data_date}' and r.status ='Success'
        group by p.project_group '''
        sql = sql.format(data_date=data_date)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        pd = self._common.mysql_select(sql, withcolumns=False)
        plt.xlim((0, 100))
        plt.ylim((0, 100))

        x_values = [15, 50, 30, 50, 65, 85,20,30,40,60]
        y_values = [20, 50, 75, 20, 75, 20,10,30,40,60]

        j = 0
        for i in pd:
            plt.scatter(x_values[j], y_values[j], s=i[1]*10, alpha=0.5)
            plt.annotate(u'{},{}'.format(i[0], i[1]), xy=(x_values[j], y_values[j]),fontsize=11)
            j = j + 1

        plt.title(u'各项目Dataflows数量(当前)', fontsize=12, color='grey', loc='left')

        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        ax.spines['bottom'].set_visible(False)

        # 去掉刻度
        ax.set_xticks([])
        ax.set_yticks([])

    def autolabel(self,ax,rects):
        """Attach a text label above each bar in *rects*, displaying its height."""
        for rect in rects:
            height = rect.get_height()
            ax.annotate('{}h'.format(int(round(height,0))),
                        xy=(rect.get_x() + rect.get_width() / 2, height),
                        xytext=(0, 3),  # 3 points vertical offset
                        textcoords="offset points",
                        ha='center', va='bottom', fontsize=11)

    def project_elapse_time(self,ax,data_date,date_before_7day):

        sql = '''select p.project_group, sum(r.elapse_time)/(60*60) as sumtime
        from dataflow2 d
        join run2 r on d.dfid = r.dfid
        join project_ext p on d.project = p.project
        where date(r.created_time) between'{date_before_7day}' and '{data_date}' and r.status ='Success'
        group by p.project_group
        order by project_group
         '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        tottime_list =self._common.mysql_select(sql, withcolumns=False)

        sql = '''
        select project_group,sum(sumtime)/7/(60*60) as sumtime
         from (
        select p.project_group,date(r.created_time) data_date,min(r.begin_time) min_time,max(r.end_time) max_time,
               timestampdiff(SECOND,min(r.begin_time),max(r.end_time)) sumtime
         from (select distinct dfid from run2 where date(created_time) = '{date_before_7day}') b -- 以7天前任务为准
         join  run2 r on b.dfid = r.dfid
         join (select dfid,date(created_time) data_date,min(created_time) as min_time 
            from run2 
           where date(created_time) between'{date_before_7day}' and '{data_date}' and status ='Success'
           group by dfid,date(created_time)
            ) m on r.dfid = m.dfid and r.created_time = m.min_time -- 近7天每天的实际耗时
         join dataflow2 df on r.dfid = df.dfid 
         join project_ext p on df.project = p.project
        group by p.project_group,date(r.created_time)
        ) t group by project_group order by project_group;
         '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        timediff_list =self._common.mysql_select(sql, withcolumns=False)
        __x_label = [row[0] for row in tottime_list]
        __y_sumtime = [float(row[1]) for row in tottime_list]
        __y_difftime = [float(row[1]) for row in timediff_list]
        __y_difftimes = [round(row, 1) for row in __y_difftime]

        __x = np.arange(len(__x_label))  # the label locations
        width = 0.4  # the width of the bars

        rects1 = ax.bar(__x - width / 2, __y_sumtime, width, label='累计耗时')
        rects2 = ax.bar(__x + width / 2, __y_difftime, width, label='实际耗时')

        ax.set_xticks(__x)
        ax.set_xticklabels(__x_label)
        ax.legend()
        self.autolabel(ax,rects1)
        self.autolabel(ax,rects2)

        # 去掉刻度
        ax.set_yticks([])
        # 隐藏边框
        ax.spines['left'].set_visible(False)
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        plt.title(u'各项目每天耗时总计', fontsize=12, color='grey', loc='left')

    def project_elapse_time_dis(self,ax,data_date,date_before_7day):

        sql = ''' select p.project_group,
                        sum(case when cast(r.elapse_time as decimal)>0 and cast(r.elapse_time as decimal)<=120 then 1 else 0 end) as cnt1,
                        sum(case when cast(r.elapse_time as decimal)>120 and cast(r.elapse_time as decimal)<=600 then 1 else 0 end) as cnt2,
                        sum(case when cast(r.elapse_time as decimal)>600 and cast(r.elapse_time as decimal)<=1800 then 1 else 0 end) as cnt3,
                        sum(case when cast(r.elapse_time as decimal)>1800 and cast(r.elapse_time as decimal)<=3600 then 1 else 0 end) as cnt4,
                        sum(case when cast(r.elapse_time as decimal)>3600 then 1 else 0 end) as cnt5
        from  run2 r
        join dataflow2 d on d.dfid = r.dfid
        join project_ext p on d.project = p.project
        where date(r.created_time) between '{date_before_7day}' and '{data_date}' and r.status='Success' and d.project is not null
        group by p.project_group '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        group_bar =self._common.mysql_select(sql, withcolumns=False)
        colors = ['#EEC591', '#EEAD0E', '#EE7600', '#EE5C42', '#EE0000']
        project_groups=[]
        for i in group_bar:

            s = float(int(i[1]) + int(i[2]) + int(i[3]) + int(i[4]) + int(i[5]))
            k = 0
            for j in range(1, 6):
                if i[0] is None:
                    p1 = plt.bar('no', float(int(i[j]))/s, bottom=k, color=colors[j - 1])

                if i[0]:
                    if int(s)==0:# 分母为0的情况
                        p1 = plt.bar(i[0], float(0), bottom=k, color=colors[j - 1])
                    else:
                        p1 = plt.bar(i[0], float(int(i[j]))/s, bottom=k, color=colors[j - 1])

                if int(i[j]) > 0:
                    plt.text(i[0], k + 0.03, u'{:.2%}'.format(int(i[j]) / s), ha='center', fontsize=11)
                if int(s)==0:# 分母为0的情况
                    k = float(0) + k
                else:
                    k = float(int(i[j]) / s) + k


        labels = [u'0-2分钟', u'2-10分钟', u'10-30分钟', u'30-60分钟', u'60分钟以上']
        mpatches = [patches.Patch(color=colors[i], label=labels[i]) for i in range(len(colors))]
        ax.legend(handles=mpatches, loc=2, bbox_to_anchor=(0.95, 1.0))

        plt.ylim((0, 1.1))

        # 去掉刻度
        ax.set_yticks([])
        # 隐藏边框
        ax.spines['left'].set_visible(False)
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)

        plt.text(-0.7, 1.1, u'各项目近7天执行时效分布图', fontsize=12, color='grey')
        # plt.title(u'\n各项目执行时效分布图',fontsize=12,color = 'grey',loc='left')

      #标题靠左显示
    def top_title(self,ax):
        # 去掉边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        ax.spines['bottom'].set_visible(False)
        # 去掉刻度
        ax.set_yticks([])
        ax.set_xticks([])
        plt.title(u'\n近7天耗时TOP10', fontsize=12, color='grey', loc='left')


    def dataflow_top(self,ax,data_date,date_before_7day):

        sql = ''' 
        select p.project_group,d.dataflow,round(r.elapse_time/60/60,1) as elapse_time
from run2 r 
join dataflow2 d on r.dfid = d.dfid
join project_ext p on d.project = p.project
where date(r.created_time) = '{data_date}' and p.project_group <> 'ODS'
order by elapse_time desc limit 10;
        '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        # d = sqlite.sqliteSelect(sql)
        d = self._common.mysql_select(sql, withcolumns=False)

        x = []
        y = []
        z= [] #所用时间
        data = []
        j = 1
        for i in d:
            a = list(i)
            a.insert(0, j)
            data.append(a)
            j = j + 1
        colors = ['#EEC591', '#EEAD0E', '#EE7600', '#EE5C42', '#EE0000']
        colors_s = ['#7B68EE','#66CD00', '#EEAD0E', '#EE0000'] #蓝色,绿色,黄色,红色
        for i in range(len(data)):
            y.append(u'{}, {}, ({})'.format(data[len(data) - 1 - i][1], data[len(data) - 1 - i][2], data[len(data) - 1 - i][0]))
            x.append(data[len(data) - 1 - i][3])
            z.append(data[len(data) - 1 - i][3])
            if data[len(data) - 1 - i][3]<=1:
                ax.barh(y[i], x[i], color='#EEC591')
            elif 1<data[len(data) - 1 - i][3]<=2:
                ax.barh(y[i], x[i], color='#EEAD0E')
            else:
                ax.barh(y[i], x[i], color='#EE0000')
            # ax.barh(y[i], x[i], color='#9F79EE')

        # 添加标注
        for xx, yy,zz in zip(x, y,z):
            plt.text(xx + 0.3, yy, u'{}h'.format(zz,float(xx) * 100), ha='center', fontsize=11)

        plt.xlim((0, float(data[0][3])+1))
        # 隐藏边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['bottom'].set_visible(False)
        # 去掉刻度
        ax.set_xticks([])

        # ax.spines['top'].set_position(('data', -1))  ##移动x轴，到y=0
        # ax.spines['left'].set_position(('data', -1))  ##还有outward（向外移动），axes（比例移动，后接小数）
        plt.title(u'\n耗时TOP10',fontsize=12,color = 'grey',loc='left')

    def scheduler_gantt(self,ax, data_date):

        # 查询任务运行时长
        sql = '''
        select lower(r.dataflow) as dataflow,min(r.begin_time), sum(r.elapse_time),case when d.project in ('edmp_ods','dl_stg') then 'ods' else d.project end as groups 
        from run2 r
        join dataflow2 d on r.dfid = d.dfid
        where date(r.created_time) = '{data_date}' and status ='Success'
        group by r.runid,r.dataflow,case when d.project in ('edmp_ods','dl_stg') then 'ods' else d.project end 
        order by min(r.begin_time) desc;
         '''
        sql = sql.format(data_date=data_date)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        # ts = sqlite.sqliteSelect(sql)
        ts = self._common.mysql_select(sql, withcolumns=False)
        # 计算最后结束时间和最先开始时间差
        sql = ''' select max(end_time),min(begin_time) from run2 where date(created_time) = '{data_date}' and status ='Success'; '''
        sql = sql.format(data_date=data_date)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        time = self._common.mysql_select(sql, withcolumns=False)

        a = time[0][0]
        b = time[0][1]

        time_a = datetime.strptime(a, '%Y-%m-%d %H:%M:%S')
        time_b = datetime.strptime(b, '%Y-%m-%d %H:%M:%S')

        t = (time_a - time_b).seconds / 3600 + 1
        t = 25 if t <= 24 else t

        task_list = [i[0] for i in ts]
        begin_time = [int(i[1].split(' ')[1].split(':')[0]) * 3600 + int(int(i[1].split(' ')[1].split(':')[1])) * 60 + int(int(i[1].split(' ')[1].split(':')[2])) for i in ts]
        elapse_time = [i[2] for i in ts]

        # xlable = [ '{}:00'.format(h%24) for h in range(t)]
        xlable = ['{}:{}'.format(h % 24, m) if m == '00' else '' for h in range(t) for m in ['00', '30']]
        x = [h * 3600 + m * 60 for h in range(t) for m in [0, 30]]

        y = []

        for i in range(len(elapse_time)):
            y.append(i)
            ax.barh(y[i] + 5, begin_time[i], color="w")
            # ax.barh(y[i] + 5, elapse_time[i], left=begin_time[i], color='blue')
            if elapse_time[i] > 2 * 60 * 60:
                ax.barh(y[i] + 5, elapse_time[i], left=begin_time[i], color='red')
                ax.annotate('{},{}'.format(ts[i][3], datetimebase.hourDesc(elapse_time[i])), xy=(begin_time[i] + elapse_time[i], y[i] + 5), fontsize=8, color='red')
            elif 1 * 60 * 60<elapse_time[i] < 2 * 60 * 60:
                ax.barh(y[i] + 5, elapse_time[i], left=begin_time[i], color='m')
                ax.annotate('{},{}'.format(ts[i][3], datetimebase.hourDesc(elapse_time[i])),xy=(begin_time[i] + elapse_time[i], y[i] + 5), fontsize=8, color='m')
            else:
                ax.barh(y[i] + 5, elapse_time[i], left=begin_time[i], color='blue')

        # ax7.plot(begin_time,y_right)
        plt.xlim((x[0], x[-1]))
        plt.ylim(0, len(elapse_time) + 5)

        ax.set_xticks(x)
        ax.set_xticklabels(xlable)
        ax.legend()
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        plt.title(u'任务执行甘特图', fontsize=12, color='grey', loc='left')
        # 去掉刻度
        ax.set_yticks([])

        # 计算每个小时同时运行的任务数量
        sql = ''' 
         select dt.bg,sum(case when r.dataflow is not null then 1 else 0 end) cnt from 
        (
        select '{date} 00:00:00' as bg,'{date} 00:30:00' as ed union all
        select '{date} 00:30:00' as bg,'{date} 01:00:00' as ed union all
        select '{date} 01:00:00' as bg,'{date} 01:30:00' as ed union all
        select '{date} 01:30:00' as bg,'{date} 02:00:00' as ed union all
        select '{date} 02:00:00' as bg,'{date} 02:30:00' as ed union all
        select '{date} 02:30:00' as bg,'{date} 03:00:00' as ed union all
        select '{date} 03:00:00' as bg,'{date} 03:30:00' as ed union all
        select '{date} 03:30:00' as bg,'{date} 04:00:00' as ed union all
        select '{date} 04:00:00' as bg,'{date} 04:30:00' as ed union all
        select '{date} 04:30:00' as bg,'{date} 05:00:00' as ed union all
        select '{date} 05:00:00' as bg,'{date} 05:30:00' as ed union all
        select '{date} 05:30:00' as bg,'{date} 06:00:00' as ed union all
        select '{date} 06:00:00' as bg,'{date} 06:30:00' as ed union all
        select '{date} 06:30:00' as bg,'{date} 07:00:00' as ed union all
        select '{date} 07:00:00' as bg,'{date} 07:30:00' as ed union all
        select '{date} 07:30:00' as bg,'{date} 08:00:00' as ed union all
        select '{date} 08:00:00' as bg,'{date} 08:30:00' as ed union all
        select '{date} 08:30:00' as bg,'{date} 09:00:00' as ed union all
        select '{date} 09:00:00' as bg,'{date} 09:30:00' as ed union all
        select '{date} 09:30:00' as bg,'{date} 10:00:00' as ed union all
        select '{date} 10:00:00' as bg,'{date} 10:30:00' as ed union all
        select '{date} 10:30:00' as bg,'{date} 11:00:00' as ed union all
        select '{date} 11:00:00' as bg,'{date} 11:30:00' as ed union all
        select '{date} 11:30:00' as bg,'{date} 12:00:00' as ed union all
        select '{date} 12:00:00' as bg,'{date} 12:30:00' as ed union all
        select '{date} 12:30:00' as bg,'{date} 13:00:00' as ed union all
        select '{date} 13:00:00' as bg,'{date} 13:30:00' as ed union all
        select '{date} 13:30:00' as bg,'{date} 14:00:00' as ed union all
        select '{date} 14:00:00' as bg,'{date} 14:30:00' as ed union all
        select '{date} 14:30:00' as bg,'{date} 15:00:00' as ed union all
        select '{date} 15:00:00' as bg,'{date} 15:30:00' as ed union all
        select '{date} 15:30:00' as bg,'{date} 16:00:00' as ed union all
        select '{date} 16:00:00' as bg,'{date} 16:30:00' as ed union all
        select '{date} 16:30:00' as bg,'{date} 17:00:00' as ed union all
        select '{date} 17:00:00' as bg,'{date} 17:30:00' as ed union all
        select '{date} 17:30:00' as bg,'{date} 18:00:00' as ed union all
        select '{date} 18:00:00' as bg,'{date} 18:30:00' as ed union all
        select '{date} 18:30:00' as bg,'{date} 19:00:00' as ed union all
        select '{date} 19:00:00' as bg,'{date} 19:30:00' as ed union all
        select '{date} 19:30:00' as bg,'{date} 20:00:00' as ed union all
        select '{date} 20:00:00' as bg,'{date} 20:30:00' as ed union all
        select '{date} 20:30:00' as bg,'{date} 21:00:00' as ed union all
        select '{date} 21:00:00' as bg,'{date} 21:30:00' as ed union all
        select '{date} 21:30:00' as bg,'{date} 22:00:00' as ed union all
        select '{date} 22:00:00' as bg,'{date} 22:30:00' as ed union all
        select '{date} 22:30:00' as bg,'{date} 23:00:00' as ed union all
        select '{date} 23:00:00' as bg,'{date} 23:30:00' as ed union all
        select '{date} 23:30:00' as bg,'{date} 24:00:00' as ed
        ) dt left join 
        (select runid,dataflow,min(begin_time) begin_time,max(end_time) end_time from run2 where date(created_time) = '{date}' and status ='Success' group by runid,dataflow) r
on r.begin_time between dt.bg and dt.ed or r.end_time between dt.bg and dt.ed or (r.begin_time<dt.bg and r.end_time>dt.ed)
         group by dt.bg
         order by dt.bg;
         '''
        sql = sql.format(date=data_date)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        task_num = self._common.mysql_select(sql, withcolumns=False)

        y2 = [i[1] for i in task_num]
        for i in range(len(x) - len(y2)):
            y2.append(0)

        ax2 = ax.twinx()
        ax2.plot(x, y2, color='y', linestyle='--')

        # 标注较大值
        for i in range(1, len(y2)):
            if y2[i] > y2[i - 1] and y2[i] > y2[i + 1] and y2[i] > 4:
                plt.annotate(y2[i], xy=(x[i], y2[i]), color='grey')


        #plt.ylim((0,12*60))
        ax2.legend(loc='best')
        # 去掉刻度
        ax2.set_yticks([])
        # 隐藏边框
        ax2.spines['top'].set_visible(False)
        ax2.spines['right'].set_visible(False)
        ax2.spines['left'].set_visible(False)

    def process_rate(self,ax):
        time_list = ['00:10', '00:20', '00:30', '00:40', '00:50', '01:00', '01:10', '01:20', '01:30', '01:40', '01:50', '02:00',
                     '02:10', '02:20', '02:30', '02:40', '02:50', '03:00', '03:10', '03:20', '03:30', '03:40', '03:50', '04:00',
                     '04:10', '04:20', '04:30', '04:40', '04:50', '05:00', '05:10', '05:20', '05:30', '05:40', '05:50', '06:00',
                     '06:10', '06:20', '06:30', '06:40', '06:50', '07:00', '07:10', '07:20', '07:30', '07:40', '07:50', '08:00',
                     '08:10', '08:20', '08:30', '08:40', '08:50', '09:00',
                     '09:10', '09:20', '09:30', '09:40', '09:50', '10:00', '10:10', '10:20', '10:30', '10:40', '10:50', '11:00',
                     '11:10', '11:20', '11:30', '11:40', '11:50', '12:00', '12:10', '12:20', '12:30', '12:40', '12:50', '13:00',
                     '13:10', '13:20', '13:30', '13:40', '13:50', '14:00', '14:10', '14:20', '14:30', '14:40', '14:50', '15:00',
                     '15:10', '15:20', '15:30', '15:40', '15:50', '16:00', '16:10', '16:20', '16:30', '16:40', '16:50', '17:00',
                     '17:10', '17:20', '17:30', '17:40', '17:50', '18:00', '18:10', '18:20', '18:30', '18:40', '18:50', '19:00',
                     '19:10', '19:20', '19:30', '19:40', '19:50', '20:00', '20:10', '20:20', '20:30', '20:40', '20:50', '21:00',
                     '21:10', '21:20', '21:30', '21:40', '21:50', '22:00', '22:10', '22:20', '22:30', '22:40', '22:50', '23:00',
                     '23:10', '23:20', '23:30', '23:40', '23:50', '24:00'
                     ]
        #查询执行各时间点执行占比
        sql = '''select * from hisen.v_hisen_run_stat'''

        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        # res = self._common.select(sql)
        result = self._common.mysql_select(sql,withcolumns = False)
        result=list(reversed(result))

        #获取x轴,y轴的值
        __x = range(0, 144)

        colors = ['grey', 'gold', 'rosybrown', 'turquoise', 'tab:olive', 'g', 'bisque', 'c', 'thistle', 'y','peru', 'darkorange', 'lightgreen']


        i = 0
        for project in result:
            point_list = [int(p) for p in project[1:]]

            if project[0]=='all':
                ax.plot(__x, point_list, color='tab:red',linewidth = '2.2',label=project[0])
                # ax.plot(__x, point_list, color='tab:red', linewidth='2', label="(" + str(i + 1) + ")" + project[0])
            elif project[0] in self._focus_project_list:
                ax.plot(__x, point_list, color='limegreen', linewidth='2',label=project[0])
            else:
                ax.plot(__x, point_list, color='lightsteelblue')

            for index, iterm in enumerate(point_list, 0):
                if point_list[index]==100:
                    x_time = time_list[index] # 算出x轴坐标时间
                    break


            if len(project[0])==5:
                plt.text(__x[len(point_list)-len([t for t in point_list if t>=max(point_list)])], max(point_list)+10, '{} {}'.format(project[0],x_time), color='indigo', rotation=60)
                # plt.text(__x[len(point_list) - len([t for t in point_list if t >= max(point_list)])],max(point_list) + 10, '{} {}%'.format(project[0], max(point_list)), color='indigo',rotation=60)
            elif len(project[0]) > 5:
                plt.text(__x[len(point_list) - len([t for t in point_list if t >= max(point_list)])],max(point_list) + 11, '{} {}'.format(project[0], x_time), color='indigo',rotation=60)
            elif len(project[0])==4:
                plt.text(__x[len(point_list) - len([t for t in point_list if t >= max(point_list)])],max(point_list) + 8, '{} {}'.format(project[0], x_time), color='indigo',rotation=60)
            elif len(project[0]) ==3 and project[0] !='all':
                plt.text(__x[len(point_list) - len([t for t in point_list if t >= max(point_list)])],max(point_list) +8, '{} {}'.format(project[0], x_time), color='indigo',rotation=60)
            elif len(project[0]) ==2:
                plt.text(__x[len(point_list) - len([t for t in point_list if t >= max(point_list)])],max(point_list) + 6, '{} {}'.format(project[0], x_time), color='indigo',rotation=60)
            elif project[0] == 'all':
                plt.text(__x[len(point_list) - len([t for t in point_list if t >= max(point_list)])],max(point_list) + 11, '{} {}'.format(project[0], x_time), color='indigo', rotation=60)

            # plt.annotate(project[0], xy=(__x[len(point_list)-len([i for i in point_list if i>=max(point_list)])],max(point_list)), xytext= None, textcoords="offset points", rotation=270)
            # plt.axvline(x = __x[len(point_list)-len([z for z in point_list if z>=max(point_list)])], c="darkgrey", ls="--", lw=1)  # 到达100，画虚线

            i+=1

        plt.ylabel(u'执行完成率(%)', fontsize=11)  # y轴标题
        # 设置x,y轴字体大小
        plt.tick_params(labelsize=11)

        # 把x轴的刻度间隔设置为1，并存在变量里
        x_major_locator = MultipleLocator(3)
        # 把y轴的刻度间隔设置为10，并存在变量里
        y_major_locator = MultipleLocator(10)
        # ax为两条坐标轴的实例
        # 把x轴的主刻度设置为1的倍数
        ax.xaxis.set_major_locator(x_major_locator)
        # 把y轴的主刻度设置为10的倍数
        ax.yaxis.set_major_locator(y_major_locator)

        # 获取24小时每30分钟时间序列
        time_list = pd.date_range(start='00:00', periods=49, freq='1800s')
        # 只保留时分秒时间序列
        short_time_list = [str(i)[11:16] for i in time_list]
        # short_time_list[-1] = '24:00'
        ax.set_xticklabels(short_time_list, rotation=70)

        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        plt.title(u'各项目调度完成时点图', fontsize=12, color='grey', loc='left')
        plt.legend(loc='lower right',fontsize=11)  # 设置折线位置，字体

    def process_rate_month(self,ax,data_date,date_before_30day):
        plt.ylim((0, 14*60))
        time_list = []
        #查询执行各时间点执行占比
        sql = '''
        select p.project_group,date(r.created_time) as data_date,max(r.created_time) as completed_time
        from (select distinct dfid from run2 where date(created_time) = '2020-10-27') b -- 以7天前任务为准
        join  run2 r on b.dfid = r.dfid
        join (select dfid,date(created_time) data_date,min(created_time) as min_time 
        from run2 
        where date(created_time) between'2020-10-27' and '2020-11-11' and status ='Success'
        group by dfid,date(created_time)
        ) m on r.dfid = m.dfid and r.created_time = m.min_time -- 每个任务以最早完成时间为准
        join dataflow2 df on r.dfid = df.dfid 
        join project_ext p on df.project = p.project
        group by p.project_group,date(r.created_time)
        order by p.project_group,date(r.created_time)
        '''
        sql = sql.format(data_date=data_date, date_before_30day=date_before_30day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        # res = self._common.select(sql)
        result = self._common.mysql_select(sql,withcolumns = False)

        __x = list(set([str(d[1]) for d in result]))
        __x.sort()

        __y = {}
        __y_label = {}
        for row in result:
            time_str = str(row[2])[11:16]
            y_value  = int(time_str.split(':')[0]) * 60 + int(time_str.split(':')[1])
            if y_value > 14*60:
                y_value = 0

            if __y.has_key(row[0]):
                __y_label[row[0]].append(time_str)
                __y[row[0]].append(y_value)
            else:
                __y_label[row[0]] = [time_str]
                __y[row[0]] = [y_value]

        # colors = ['grey', 'gold', 'rosybrown', 'turquoise', 'tab:olive', 'g', 'bisque', 'c', 'thistle', 'y','peru', 'darkorange', 'lightgreen']
        # i = 0
        # for pname, pvalue in __y.items():
        #     if pname in self._focus_project_list:
        #         plt.plot(__x, pvalue, color='lightgreen', linewidth='1', label=pname)
        #         for p in range(len(pvalue)):
        #             plt.text(__x[p], pvalue[p], '{}'.format(__y_label[pname][p]), color='indigo', rotation=60)
        #     else:
        #         plt.plot(__x, pvalue, color='lightsteelblue', linewidth='1', label=pname)
        #     i +=1


        print('__y',__y)
        print('__ylabel',__y_label)
        print('__x',__x)
        # print('__xlabel',__xlabel)
        s = 1  # 每个点大小
        rotations = [0]

        # for k, v in __y.items(): # 去除超过14点的点
        #     for b in __y[k]:
        #         if b > 840:
        #             __y[k].remove(b)

        for key, v in __y.items():
            if key in self._focus_project_list:

                for r,c in zip(__x,__y[key]):
                    if c>0:
                        plt.scatter(r, c, s = s*85, c = self._focus_color_mapping['浑天'], alpha = 1)
                for a, b in zip(__x, __y[key]):
                      if b>0:
                            plt.text(a, b, u'{}'.format(key), color=self._focus_color_mapping['浑天'])
            else:
                ro = random.choice(rotations)
                for r,c in zip(__x,__y[key]):
                    if c>0:
                        plt.scatter(r, c, s = s*20, c = self._focus_color_mapping['other'], alpha = 0.8)
                for a, b in zip(__x, __y[key]):
                    if b > 0:
                        plt.text(a, b, u'{}'.format(key), color=self._focus_color_mapping['other'],rotation=ro)
        ax.set_xticklabels(__x, rotation=70)
        ax.axhline(y=540, c="r", ls="--", lw=1)
        ax.text(0, 540, '{}'.format('09:00'), color='r')




        # 设置x,y轴字体大小
        plt.tick_params(labelsize=11)

        # 把x轴的刻度间隔设置为1，并存在变量里
        #x_major_locator = MultipleLocator(1)
        # 把y轴的刻度间隔设置为10，并存在变量里
        #y_major_locator = MultipleLocator(1)
        # ax为两条坐标轴的实例
        # 把x轴的主刻度设置为1的倍数
        #ax.xaxis.set_major_locator(x_major_locator)
        # 把y轴的主刻度设置为10的倍数
        #ax.yaxis.set_major_locator(y_major_locator)

        # 获取24小时每30分钟时间序列
        time_list = pd.date_range(start='00:00', periods=49, freq='1800s')
        # 只保留时分秒时间序列
        short_time_list = [str(i)[11:16] for i in time_list]
        # short_time_list[-1] = '24:00'
        ax.set_xticklabels(__x, rotation=70)

        ax.set_yticks([])

        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        plt.title(u'各项目近30天完成时点图', fontsize=12, color='grey', loc='left')
        plt.legend(loc='lower right',fontsize=11)  # 设置折线位置，字体

    def project_elapse_time_everyday(self,ax,data_date,date_before_7day):

        sql = '''
        -- 最近七天资源不足导致的任务失败数
        select dd.data_date,count(1) cnt
        from
        (
          select date('2020-12-04') as data_date union all
          select date('2020-12-04') + interval 1 day as data_date union all
          select date('2020-12-04') + interval 2 day as data_date union all
          select date('2020-12-04') + interval 3 day as data_date union all
          select date('2020-12-04') + interval 4 day as data_date union all
          select date('2020-12-04') + interval 5 day as data_date union all
          select date('2020-12-04') + interval 6 day as data_date
        ) dd
        left join
        (
        select dfid,dataflow,date(created_time) as data_date,
               min(case when status ='Success' then created_time else null end) as suc_time,
               max(case when status = 'Failed' then created_time else null end) as fail_time
         from run 
         where date(created_time) >= '2020-12-04'
         group by dfid,dataflow,date(created_time)
         )t on dd.data_date = t.data_date and TIMESTAMPDIFF(SECOND,t.fail_time,t.suc_time) between 0 and 5*60
         group by dd.data_date
         order by dd.data_date;
         '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        pro_list =self._common.mysql_select(sql, withcolumns=False)

        x = [i[0] for i in pro_list]
        y = [i[1] for i in pro_list]
        plt.bar(x, y, color=['#9F79EE', '#FA8072', '#5CACEE', 'greenyellow', 'lightsalmon', 'fuchsia', 'aqua', 'springgreen', 'orange'], tick_label=x)
        # 设置数字标签

        for a, b in zip(x, y):
            k = str(b)
            plt.text(a, float(b), u'{}'.format(k), ha='center', va='bottom', fontsize=11)

        # plt.title(u'各项目每天执行累加耗时', fontsize=12, color='grey', loc='left')
        plt.title(u'最近七天资源不足导致的任务失败数', fontsize=12, color='grey', loc='left')
        # 去掉边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        # 去掉刻度

        # ax.set_yticks([20 if max(y)>20 else 10])
        if max(y)>20:
            ax.set_ylim([0, 20])
        else:
            ax.set_ylim([0, 10])
    def milepost_week(self,ax,data_date,date_before_7day):
        # 浑天里程碑执行点
        sql = '''
            select date(r.created_time) data_date,
            min(r.begin_time) as begin_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_cust_extend' then r.end_time else null end) as extend_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_mapping_customer' then r.end_time else null end) as mapping_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_suspected_customer' then r.end_time else null end) as suspected_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_merge_customer' then r.end_time else null end) as merge_time,
            max(r.end_time) as all_complete_time
            from (select distinct dfid from run2 where date(created_time) = '{data_date}') b -- 以7天前任务为准
            join  run2 r on b.dfid = r.dfid
            join (select dfid,date(created_time) data_date,min(created_time) as min_time 
            from run2 
            where date(created_time) between'{date_before_7day}' and '{data_date}' and status ='Success'
            group by dfid,date(created_time)
            ) m on r.dfid = m.dfid and r.created_time = m.min_time -- 近7天每天的实际耗时
            join dataflow2 df on r.dfid = df.dfid 
            where df.project = 'ecif'
            group by date(r.created_time)
            order by date(r.created_time) desc
         '''
        sql = sql.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql, func=sys._getframe().f_code.co_name)
        pro_list =self._common.mysql_select(sql, withcolumns=False)
        result=pro_list
        # print pro_list
        sql_1="""
            -- 浑天30天平均里程碑时点
            select max(data_date) + interval 1 day,
            round(sum(TIMESTAMPDIFF(MINUTE, data_date, begin_time))/7,0) as begin_time,
            round(sum(TIMESTAMPDIFF(MINUTE, data_date, extend_time))/7,0) as extend_time,
            round(sum(TIMESTAMPDIFF(MINUTE, data_date, mapping_time))/7,0) as mapping_time,
            round(sum(TIMESTAMPDIFF(MINUTE, data_date, suspected_time))/7,0) as suspected_time,
            round(sum(TIMESTAMPDIFF(MINUTE, data_date, merge_time))/7,0) as merge_time,
            round(sum(TIMESTAMPDIFF(MINUTE, data_date, all_complete_time))/7,0) as all_complete_time
            from (
            select date(r.created_time) data_date,
            min(r.begin_time) as begin_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_cust_extend' then r.end_time else null end) as extend_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_mapping_customer' then r.end_time else null end) as mapping_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_suspected_customer' then r.end_time else null end) as suspected_time,
            min(case when r.dataflow = 'etl.huting_ecif.csr_merge_customer' then r.end_time else null end) as merge_time,
            max(r.end_time) as all_complete_time
            from (select distinct dfid from run2 where date(created_time) = '{date_before_7day}') b -- 以7天前任务为准
            join  run2 r on b.dfid = r.dfid
            join (select dfid,date(created_time) data_date,min(created_time) as min_time
            from run2
            where date(created_time) between'{date_before_7day}' and '{data_date}' and status ='Success'
            group by dfid,date(created_time)
            ) m on r.dfid = m.dfid and r.created_time = m.min_time -- 近7天每天的实际耗时
            join dataflow2 df on r.dfid = df.dfid 
            where df.project = 'ecif'
            group by date(r.created_time)
            )t
        """
        sql_1 = sql_1.format(data_date=data_date, date_before_7day=date_before_7day)
        self._logbase.info(sql_1, func=sys._getframe().f_code.co_name)
        result2 = self._common.mysql_select(sql_1, withcolumns=False)
        # print('result2',result2)
        result2 = result2[0]
        __color = ['b', 'tab:brown', 'k', 'g', 'm', 'r', 'y']
        __colors = ['lightsteelblue', 'lightcyan', 'lightgrey', 'palegreen', 'lightpink', 'mistyrose', 'lightyellow']
        time_line_list = ['begin', 'extend_time', 'mapping_time', 'suspected_time', 'merge_time', 'all_complete_time']
        time_line_list_name = [u'①起调', u'②宽表完成', u'③Mapping表完成', u'④归并完成', u'⑤Merge表完成', u'⑥全部完成']
        num_list = ['①','②','③','④','⑤','⑥']
        def num2HourMinSec(num):
            """转为小时分钟"""
            m, s = divmod(num, 60)
            h, m = divmod(m, 60)
            timestr = "%02d:%02d" % (h, m)
            return timestr

        __x_date = {}
        __x_line = {}
        for row in result:
            __x_date[int(str(row[0]).replace('-', ''))] = [int(time_str.split(':')[0]) * 60 + int(time_str.split(':')[1]) for time_str in [dd.split(' ')[1] for dd in row[1:]]]

        for __y, __x in __x_date.items():
            ax.plot(__x, [__y for i in range(len(__x))], color='tab:blue')
            i = 0
            for __p in __x:
                plt.scatter(__p, __y, s=40, alpha=0.5, c=__color[i])
                plt.text(__p+1, __y+0.15,u'{} {}'.format(num_list[i],num2HourMinSec(__p*60)), color=__color[i])
                i += 1

        for i in range(len(time_line_list)):
            #ax.plot([result2[i + 1], result2[i + 1]],
                    #[int(str(min(__x_date.keys())).replace('-', '')) - 0.1, int(str(result2[0]).replace('-', ''))],
                    # ls="--", color=__colors[i])
            if time_line_list_name[i] == u'Mapping表完成':
                plt.text(result2[i + 1] - 0, int(str(result2[0]).replace('-', '')),
                         u'{}'.format(time_line_list_name[i]), color=__color[i])
            elif time_line_list_name[i] == u'起调':
                plt.text(result2[i + 1] +1, int(str(result2[0]).replace('-', '')),
                         u'{}'.format(time_line_list_name[i]), color=__color[i])
            elif time_line_list_name[i] == u'宽表完成':
                plt.text(result2[i + 1] - 0, int(str(result2[0]).replace('-', '')),
                         u'{}'.format(time_line_list_name[i]), color=__color[i])
            elif time_line_list_name[i] == u'归并完成':
                plt.text(result2[i + 1] - 0, int(str(result2[0]).replace('-', '')),
                         u'{}'.format(time_line_list_name[i]), color=__color[i])
            elif time_line_list_name[i] == u'Merge表完成':
                plt.text(result2[i + 1] - 0, int(str(result2[0]).replace('-', '')),
                         u'{}'.format(time_line_list_name[i]), color=__color[i])
            else:
                plt.text(result2[i + 1] - 1, int(str(result2[0]).replace('-', '')),
                         u'{}'.format(time_line_list_name[i]), color=__color[i])

        __yticks = __x_date.keys()
        __yticks.sort()

        __yticksstr = [str(n) for n in __yticks]

        ax.set_yticks(__yticks)
        ax.set_yticklabels(__yticksstr,rotation=360)
        ax.set_ylim([min(__yticks) - 1, max(__yticks) + 2])
        for i in __yticks:
            ax.text(0, i, '{}'.format('{}'.format(str(i))), color='tab:blue',fontsize=10)

        __x_xticks = [i * 10 for i in range(12 * 60 / 10)]
        __x_xticklables = [p if p.split(':')[1] in ['00', '30'] else '' for p in
                           ['{}'.format(num2HourMinSec(s * 60)) for s in __x_xticks]]
        ax.set_xlim([0, 12*60])
        ax.set_xticks(__x_xticks)
        ax.set_xticklabels(__x_xticklables)

        # ax.yaxis.set_ticks_position()
        # from matplotlib.ticker import FuncFormatter
        # to_percent=0.85
        # ax.yaxis.set_major_formatter(FuncFormatter(to_percent))
        # ax.xaxis.set_major_formatter(FuncFormatter(to_percent))
        plt.title(u'浑天里程碑执行点', fontsize=12, color='grey', loc='left')
        # 去掉边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        # 去掉刻度
        ax.set_yticks([])


    def figure_blank_space(self,ax,data_date):
        # 设置空白，为了第7图显示
        plt.xlim((0, 20))
        plt.ylim((0, 5))
        # 去掉边框
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.spines['left'].set_visible(False)
        ax.spines['bottom'].set_visible(False)
        # 去掉刻度
        ax.set_xticks([])
        ax.set_yticks([])
if __name__ == '__main__':
    main()
