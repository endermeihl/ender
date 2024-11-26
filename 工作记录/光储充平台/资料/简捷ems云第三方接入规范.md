
# 变更记录
|    日期     |版本|  提交  |                                 内容                                  | 备注 |
| ---------- |--| ------ | -------------------------------------------------------------------- | ---- |
| 2024-08-16 | 1.0 |张国宏 |                初始版本                             |      |
| 2024-08-21 | 1.1 |张国宏 |                修订topic前缀                           |      |

# 前言
## 介绍
定义EMS控制器(EMS2.0) 与EMMS云(emmsv2)通信标准和规范
同样的业务有两种topic , 一种为payload为非压缩格式, 另外一种为前者叠加lz4算法后的byte流
平台和设备都需要支持非压缩和lz4压缩业务,推荐使用lz4
## 点位定义
每个点位标识确定含义,确定的单位
不同设备的状态点位,枚举值不一样时,不能用同一个点位
## 术语
LC_SN:EMS控制器编号,全局唯一,产品铭牌上的SN
CB_ID:EMS柜编号,租户下唯一,选填,客户资产管理的凭据
Sub_ID:南向设备编号,一个控制器下唯一,用于区分控制器设备
FUNC_ID:功能代码
LZ4:压缩算法,用于原始报文压缩报文后缀
LZ4_LEN:压缩前内容长度,长报文必须增加,格式十进制如120
tenant_id:租户ID
## 通讯规范
### 控制器主动上报,平台不应答
- 控制器数据上报
- 告警上报
- 心跳

上行 topic
```
非压缩: third/[tenant_id]/emms2//LcPost/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//LcPost/[LC_SN]/[FUNC_ID]/lz4
```

### 控制器主动上报,平台应答
- 设备登录

上行 topic
```
非压缩: third/[tenant_id]/emms2//LcPost/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//LcPost/[LC_SN]/[FUNC_ID]/lz4/[LZ4_LEN]
```
下行 topic
```
非压缩: third/[tenant_id]/emms2//LcPostResp/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//LcPostResp/[LC_SN]/[FUNC_ID]/lz4/[LZ4_LEN]
```

### 平台召测,设备响应
- 设备配置表查询
- 控制器参数查询
- 告警同步

下行 topic
```
非压缩: third/[tenant_id]/emms2//Get/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//Get/[LC_SN]/[FUNC_ID]/lz4/[LZ4_LEN]
```
上行 topic
```
非压缩: third/[tenant_id]/emms2//GetResp/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//GetResp/[LC_SN]/[FUNC_ID]/lz4/[LZ4_LEN]
```
### 平台设置,设备应答
- 控制指令下发

下行 topic
```
非压缩: third/[tenant_id]/emms2//Set/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//Set/[LC_SN]/[FUNC_ID]/lz4/[LZ4_LEN]
```
上行 topic
```
非压缩: third/[tenant_id]/emms2//SetResp/[LC_SN]/[FUNC_ID]
lz4压缩: third/[tenant_id]/emms2//SetResp/[LC_SN]/[FUNC_ID]/lz4/[LZ4_LEN]
```

### pyload格式

### 数据包基础字段
```json
{
  "funcId":"[FUNC_ID]",
  "lcSN": "[LC_SN]",
  "seq": 150,
  "time":1662002513
}
```

### 扩展字段
响应类
```json
{
  "result":0
}
```
南向采集上报/响应类
```json
{
 
}
```
> 谁维护TAG的转换表格

## 业务接口
### 控制器登录
重启后发登录,成功后发送心跳
如果云端添加了设备到某个租户,result 返回0,未添加返回0
上行 topic:    third/[tenant_id]/emms2//LcPost/21881E000183/Login
上行 payload:
```json
{
  "funcId":"Login",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "info":{
        "protocolVer":"1.0.0",                //1.0.0 初始版本
        "location":"90.12,123",         //"坐标",       
        "capacity":200,                //"容量",       
        "power":100,                   //"功率",                     
        "station_name": "宏程总装车间",   //"站名", 
        "vendor":"LNXALL",         //"EMS控制器厂家",   
        "softVersion":"1.0.1000",  //控制器软件版本
        "tenant_id":"000000",    //租户ID
        "model":"EM1000"   //"控制器型号",        
  }
}
```
> id 用于记录柜编号
> station_name 记录当前站的名称,一旦部署不建议修改,否则将导致用户数据丢失
> tenant_id 用于大客户定制分配租户,不传或者000000将添加置系统管理员租户000000中
> location 十进制度制是WGS84坐标最常用的表示方法。它将经度和纬度表示成十进制数，小数点后面的数字表示距离地球赤道或本初子午线的距离(单位为度)。例如，北京的WGS84坐标可以表示
为“116.40,39.90”。
下行 topic:    third/[tenant_id]/emms2//LcPostResp/21881E000183/Login
下行 payload:
```json
{
  "funcId":"Login",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```

### 控制器心跳(1min)

上行 topic:    third/[tenant_id]/emms2//LcPost/21881E000183/HeartBeat
上行 payload:

```json
{
  "funcId":"HeartBeat",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "info":
  {
    "alarmCnt":0
  }
}
```

下行 topic:    third/[tenant_id]/emms2//LcPostResp/21881E000183/HeartBeat
下行 payload:

```json
{
  "result":0,
  "lcSN":"21881E000183",
  "time":1662002513,
  "funcId":"HeartBeat",
  "seq":126
}
```

### 控制器设备表查询
登录成功后上报一次
上行 topic:    third/[tenant_id]/emms2//LcPost/21881E000183/DeviceInfo
上行 payload:
```json
{
  "funcId":"DeviceInfo",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "info":
  {
      "dev_class":[
            {
              "type":"METER",
              "devs":[
                  {"no":"PCS_METER","type":"PCS","name":"储能表","addr":"305195410001","template":"METER_COMMON_DLT645.json","vendor": "宁波三星","model": "METER-SANXING-DTZY188",},
                  {"no":"GRID_METER","type":"GRID","name":"关口表","addr":"104263000107","template":"METER_COMMON_DLT645.json","vendor": "宁波三星","model": "METER-SANXING-DTZY188",}
              ]
          },
          {
              "type":"PCS",
              "devs":[
                  {"group":"LC1", "no":"LC1.PCS","name":"逆变器","addr":"1","template":"PCS_GSSTES_MA1000K0100.json","vendor": "IN_POWER","model": "INP_PCS_100_SPLIT_PHASE",},
                  {"group":"LC2", "no":"LC2.PCS","name":"逆变器","addr":"1","template":"PCS_GSSTES_MA1000K0100.json","vendor": "IN_POWER","model": "INP_PCS_100_SPLIT_PHASE",}
              ]
          }
      ]
  }
}
```
> "container":对等模式不会上传,用于集装箱系统中区分控制器所属关系,非集装箱模式下不用上传
> "group":对等模式不会上传,用于在主从柜模型中区分控制器下分组关系,

下行 topic:    third/[tenant_id]/emms2//LcPostResp/21881E000183/DeviceInfo
下行 payload:

```json
{
  "result":0,
  "lcSN":"21881E000183",
  "time":1662002513,
  "funcId":"DeviceInfo",
  "seq":126
}
```

       
### 控制器策略数据读取

> 云端没从需求

### 控制器策略数据写入
下行 topic:    third/[tenant_id]/emms2//Set/21881E000183/Schedule
下行 payload:
```json
{
  "funcId":"Schedule",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "messages": [
  {
    "Date":"2023-02-01",
    "Schedule":[
        {
            "StartTime":"00:00",
            "EndTime":"00:30",
            "Power":100 
        },   
        {
            "StartTime":"01:00",
            "EndTime":"01:30",
            "Power":-100 
        }    
    ]
  } 
  ]        
}
```
//> info.type //长期计划不判断时间,并且直接取messages的第一个,不支持多个,短期计划,最多支持1年

### 控制器策略数据写入
下行 topic:    third/[tenant_id]/emms2//Set/21881E000183/Schedule
下行 payload:
```json
{
  "funcId":"Schedule",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "messages": [
  {
    "Date":"2023-02-01",
    "Schedule":[
        {
            "StartTime":"00:00",
            "EndTime":"00:30",
            "Power":100 
        },   
        {
            "StartTime":"01:00",
            "EndTime":"01:30",
            "Power":-100 
        }    
    ]
  } 
  ]        
}
```


上行 topic:    third/[tenant_id]/emms2//SetResp/21881E000183/Schedule
上行 payload:
```json
{
  "funcId":"Schedule",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result":0
}
```

----

### 控制器数据表
```json
{
    //第一部分（文本配置参数 不会周期上报）
            "EnTraceLoad":0,             //使能负荷跟踪
            "EnProtectReverse":0,        //使能逆流防护
            "EnProtectTransf":0,         //使能变压器需量
            "EnProtectDOD":0,            //使能DOD防护
            "ControlMode":0,             //设置远程/本地模式
            "Pmlmax":0,                  //变压器容量
            "K1":0,                      //需量控制系数
            "K2":0,                      //负载跟踪系数
            "SOCmax":0,                  //最大充电SOC
            "SOCmin":0,                  //最小放电SOC
            "Pset":0,                    //防逆流余量
            "Adjpreriod":0,              //调整间隔(秒)
            "TempThreshold":0,           //温度阈值
            "LoadTrackingLimit":0,       //使能负荷跟踪限制
            "DemandThreshold1":0,        //1月需量控制门限
            "DemandThreshold2":0,        //2月需量控制门限
            "DemandThreshold3":0,        //3月需量控制门限
            "DemandThreshold4":0,        //4月需量控制门限
            "DemandThreshold5":0,        //5月需量控制门限
            "DemandThreshold6":0,        //6月需量控制门限
            "DemandThreshold7":0,        //7月需量控制门限
            "DemandThreshold8":0,        //8月需量控制门限
            "DemandThreshold9":0,        //9月需量控制门限
            "DemandThreshold10":0,       //10月需量控制门限
            "DemandThreshold11":0,       //11月需量控制门限
            "DemandThreshold12":0,       //12月需量控制门限
            "DemandErr":0,               //最大需量偏差
            "DemandStartHour":0,         //需量充电开始小时
            "DemandStartMin":0,          //需量充电开始分钟
            "DemandEndHour":0,           //需量充电结束小时
            "DemandEndMin":0,            //需量充电结束分钟
            "StratagyType":0,            //设置策略模式  0:削峰填谷模式 1:变压器增容 2:手动输入 3:光储联动 
            "EnBmsAlarm":0,             //使能BMS告警
            "EnDraughtFanPower":0,       //使能风机跟随功率
            "DFanPowerThreShold":0,      //风机跟随功率门限
            "EnDraughtFanTem":0,         //使能风机跟随温度
            "DFanTemThreShold":0,        //风机跟随温度门限
            "DeadTime":0,                //跟随时间死区
            "MeterInvert":0,             //储能表方向取反
            "EnIoProtect":0,      //使能动环保护
            "PcsAutoTurn":0,             //自动PCS关机
            "PcsAutoTurnOffTime":0,      //自动PCS关机时间(秒)
            "AirCondAheadTime":30,     // 空调提前时间(分钟)
            "AirCondDelayTime":0,      // 空调延迟时间(分钟)
			"chargeMode":0,          	//充电模式 1固定功率 2预定计划
			"dischargeMode":0,       	//放电模式 1固定功率 2预定计划 3负载跟踪
			"supplyPowerDownError":0, //下限误差
			"kBefor":0,    		  			//空调提前开启时间，没设置默认是0
			"kAfter":0,              	//空调延后关闭时间，没设置默认是0
			"singleMaxVoltage":0,     //最高单体电压,支持小数
			"singleMinVoltage":0,     //最低单体电压，支持小数
			"chargeMaxPower":0,       //PCS最大充电功率 没设置默认是0
			"dischargeThreshold":0,   //Pcs最大放电功率 没设置默认是0

 //第二部分（可以设置参数 不会周期上报）
            "OnOff":0,            //代理PCS开关机 0：关机 1：开机
            "SetPower":0,         //设置有功功率
            "SetRePower":0,       //设置无功功率


//第三部分（只读参数 会周期上报）



}
----
### 控制器遥测（只读）数据上报
周期上报(5min) ，可配置上报周期。
上行 topic:   third/[tenant_id]/emms2//LcPost/21881E000183/Telemetry
上行 payload:
```json
{
  "funcId":"Telemetry",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "tags": {
      // 参数是 控制器数据表 第三部分的参数   格式如下：
            "SysStatus":0,        //系统运行状态 0正常 1异常
            "ExpectPower":0,      //期望有功功率
            "GridPower":0,        //电网功率
            "LoadPower":0,        //负载功率
            "LocalCtrlPower":0,   //LC输入功 
            "SysStatus":0,        //系统运行状态 0正常 1异常
            "ExpectPower":0,      //期望有功功率
            "GridPower":0,        //电网功率
            "LoadPower":0,        //负载功率
            "LocalCtrlPower":0,   //LC输入功率
            "CabPower":0,         //额定功率
            "CabCapacity":0,      //额定容量
            "Income":0,           //今日收益
            "Efficiency":0,       //今日效率
            "ControlMode":0,        //控制模式 0:远程模式(接收上游命令) 1:本地模式(屏幕/上位机控制)
            "StratagyType":0,       //策略模式  0:削峰填谷模式 1:变压器增容 2:手动输入 3:光储联动            
    }         
}
```
下行 topic:    third/[tenant_id]/emms2//LcPostResp/21881E000183/Telemetry
下行 payload:
```json
{
  "funcId":"Telemetry",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```


### 控制器遥控数据写入
下行 topic:    third/[tenant_id]/emms2//Set/21881E000183/Telecontrol
下行 payload:
```json
{
  "funcId":"Telecontrol",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
   "tags": {
              //支持 控制器数据表 中的所有点位 格式如下：
              "OnOff":0,            //代理PCS开关机 0：关机 1：开机
              "SetPower":0,         //设置有功功率
              "SetRePower":0,       //设置无功功率
              "ClrAlarm":0,         //清除保护

              "EnTraceLoad":0,             //使能负荷跟踪
              "EnProtectReverse":0,        //使能逆流防护
              "EnProtectTransf":0,         //使能变压器需量
              "EnProtectDOD":0,            //使能DOD防护
              "ControlMode":0,             //设置远程/本地模式
              "Pmlmax":0,                  //变压器容量
              "K1":0,                      //需量控制系数
              "K2":0,                      //负载跟踪系数
              "SOCmax":0,                  //最大充电SOC
              "SOCmin":0,                  //最小放电SOC
              "Pset":0,                    //防逆流余量
              "Adjpreriod":0,              //调整间隔(秒)

              "PcsAutoTurn":0,             //自动PCS关机
              "PcsAutoTurnOffTime":0,      //自动PCS关机时间(秒)           
   }
}  
```

上行 topic:    third/[tenant_id]/emms2//SetResp/21881E000183/Telecontrol
上行 payload:
```json
{
  "funcId":"Telecontrol",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```

----

### 南向设备遥测数据上报
周期上报
上行 topic:   third/[tenant_id]/emms2//LcPost/21881E000183/SubTelemetry
上行 payload:
```json
{
  "funcId":"SubTelemetry",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
   "messages":[
    {
        "no": "METER", //电表METER
        "tags": {
            /*--------------通用------------------*/
            "Online":0,                //"在线状态"
            "ActiveEnergy":0,          //"正向有功总电能"
            "ReactiveEnergy":0,        //"反向有功总电能"
            "AphaseVoltage":0,         //"A相电压"      
            "BphaseVoltage":0,         //"B相电压"      
            "CphaseVoltage":0,         //"C相电压"      
            "AphaseCurrent":0,         //"A相电流"      
            "BphaseCurrent":0,         //"B相电流"     
            "CphaseCurrent":0,         //"C相电流"
            "ActivePower":0,           //"瞬时总有功功率"
            "AphaseActivePower":0,     //"瞬时A相有功功率
            "BphaseActivePower":0,     //"瞬时B相有功功率
            "CphaseActivePower":0,     //"瞬时C相有功功率
            "ReactivePower":0,         //"瞬时总无功功率"
            "AphaseReactivePower":0,   //"瞬时A相无功功率
            "BphaseReactivePower":0,   //"瞬时B相无功功率
            "CphaseReactivePower":0,   //"瞬时C相无功功率
            "ApparentPower":0,         //"瞬时总视在功率"
            "AphaseApparentPower":0,   //"瞬时A相视在功率
            "BphaseApparentPower":0,   //"瞬时B相视在功率
            "CphaseApparentPower":0,   //"瞬时C相视在功率
            "PowerFactor":0,           //"总功率因素"
            "AphasePowerFactor":0,     //"A相功率因素
            "BphasePowerFactor":0,     //"B相功率因素
            "CphasePowerFactor":0,      //"C相功率因素

            /*------------ANTING,AT182G-CT------------------*/
            "PositiveActiveRate1":0,            //费率1正向有功电量 
            "PositiveActiveRate2":0,            //费率2正向有功电量 
            "PositiveActiveRate3":0,            //费率3正向有功电量 
            "PositiveActiveRate4":0,            //费率4正向有功电量 
            "ReverseActiveRate1":0,             //费率1反向有功电量 
            "ReverseActiveRate2":0,             //费率2反向有功电量 
            "ReverseActiveRate3":0,             //费率3反向有功电量 
            "ReverseActiveRate4":0,             //费率4反向有功电量 
            /*------------SANXING,DTZY188------------------*/
            "PositiveActiveRate1":0,            //费率1正向有功电量 
            "PositiveActiveRate2":0,            //费率2正向有功电量 
            "PositiveActiveRate3":0,            //费率3正向有功电量 
            "PositiveActiveRate4":0,            //费率4正向有功电量 
            "ReverseActiveRate1":0,             //费率1反向有功电量 
            "ReverseActiveRate2":0,             //费率2反向有功电量 
            "ReverseActiveRate3":0,             //费率3反向有功电量 
            "ReverseActiveRate4":0,             //费率4反向有功电量 
            /*------------ZHENGTAI,DTSD666------------------*/
            "PositiveActiveRate1":0,            //费率1正向有功电量 
            "PositiveActiveRate2":0,            //费率2正向有功电量 
            "PositiveActiveRate3":0,            //费率3正向有功电量 
            "PositiveActiveRate4":0,            //费率4正向有功电量 
            "ReverseActiveRate1":0,             //费率1反向有功电量 
            "ReverseActiveRate2":0,             //费率2反向有功电量 
            "ReverseActiveRate3":0,             //费率3反向有功电量 
            "ReverseActiveRate4":0,             //费率4反向有功电量 
            "PositiveReactiveEnergy":0,      //正向无功总电能    
            "PositiveReactiveRate1":0,       //正向无功费率1电能 
            "PositiveReactiveRate2":0,       //正向无功费率2电能 
            "PositiveReactiveRate3":0,       //正向无功费率3电能 
            "PositiveReactiveRate4":0,       //正向无功费率4电能 
            "ReverseReactiveEnergy":0,       //反向无功总电能 
            "ReverseReactiveRate1":0,        //反向无功费率1电能 
            "ReverseReactiveRate2":0,        //反向无功费率2电能 
            "ReverseReactiveRate3":0,        //反向无功费率3电能 
            "ReverseReactiveRate4":0,        //反向无功费率4电能 
            "Frequency":0                   //频率

        }        
    },
    {
        "no": "AIRCOND", //空调
        "tags": {
          /*--------------通用------------------*/
            "Online":0,             //在线状态
            "CondenserTemp":0,     //冷凝温度 
            "IndoorTemp":0,        //室内温度 
            "Humidity":0,          //湿度 
            "OnOff":0,             //开关机
            "CompressorSta":0,     //压缩机状态
            "IntFanSta":0,         //内风机状态
            "OutFanSta":0,         //外风机状态
        }        
    },    {
        "no": "LIQUID_COOL", //液冷机
        "tags": {
          /*--------------通用------------------*/
            "Online":0,             //在线状态
            "UnitSta":0,//整机状态
            "Pump":0,     //水泵状态 
            "CompressorSta":0,        //压缩机状态 
            "HeaterSta":0,          //电加热状态 
            "EffluentTemp":0,             //出水温度
            "InflowTemp":0,     //入水温度
            "OutdoorTemp":0,         //环境温度
        }        
    },
    {
        "no": "BMS", //BMS数据
        "tags": {
          /*--------------通用------------------*/
              "Online":0,                    //在线状态
              "SOC":0,                      //RackSOC                    
              "SOH":0,                      //RackSOH                                 
              "RackRunState":0,              //Rack运行状态  
              "RackPosInsulatVal":0,         //电池簇绝缘电阻R+
              "RackNegInsulatVal":0,         //电池簇绝缘电阻R-
              "RackVoltage":0,               //Rack电池总电压 
              "RackCurrent":0,               //Rack电池总电流 
              "RackMinTemp":0,                 //电池最低温度          
              "RackMaxTemp":0,                 //电池最高温度  
              "RackAverageTemp":0,             //电池平均温度
              "RackAverageVolt":0,             //单体平均电压 单位mV
              "RackMinVoltage":0,              //最低单体电压 单位mV
              "RackMaxVoltage":0,              //最高单体电压 单位mV

              "CellVol": [3000,3000,3000,3000,3000], //单位mV
              "CellTemp": [10.0,11.0,12.5,11,10.5], //温度采样点
              "PoleTemp": [10.0,11.0,12.5,11,10.5], //极柱温度,按照+,-,+,-的原则上报
        }        
    },
    {
        "no": "PCS", //PCS数据
        "tags": {
          /*--------------通用------------------*/
            "Online":0,               //在线状态
            "ActivePower":0,          //有功功率          
            "ReactivePower":0,        //无功功率
            "ApparentPower":0,        //视在功率
            "AphaseVoltage":0,        //A相电压         
            "BphaseVoltage":0,        //B相电压         
            "CphaseVoltage":0,        //C相电压         
            "AphaseCurrent":0,        //A相电流         
            "BphaseCurrent":0,        //B相电流         
            "CphaseCurrent":0,        //C相电流         
            "Frequency":0,           //电网频率              
        }
    },
    {
      "no": "DIDO", //DIDO
       "tags": {
          /*--------------通用------------------*/
          "Online":0,             //在线状态
        }
    },
    {
      "no": "HUMITURE", 
       "tags": {
          /*--------------通用------------------*/
          "Online":0,             //在线状态   
          "temperature":0, //温度  
          "humidity":0 //湿度  
        }
    }
  ]
}
```
> no:设备SN, 对等模式一般为PCS;主从模式一般是LC1.PCS,电表仍然是GRID_METER

下行 topic:    third/[tenant_id]/emms2//LcPostResp/21881E000183/SubTelemetry
下行 payload:
```json
{
  "funcId":"SubTelemetry",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```

### 南向设备遥测数据查询
下行 topic:    third/[tenant_id]/emms2//Get/21881E000183/SubTelemetry
下行 payload:
```json
{
  "funcId":"SubTelemetry",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
    "messages":[
    {
        "no": "[Sub_ID]",
    },
    {
        "no": "[Sub_ID]",
    }
    ]
}  
```
上行 topic:   third/[tenant_id]/emms2//GetResp/21881E000183/SubTelemetry
上行 payload:同南向设备遥测数据上报

### 南向设备遥控数据写入
下行 topic:    third/[tenant_id]/emms2//Set/21881E000183/SubTelecontrol
下行 payload:
```json
{
  "funcId":"SubTelecontrol",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "messages":[
   {
        "no": "[Sub_ID]",
        "tags": {
            
            }
  }
  ]
}  
```

上行 topic:    third/[tenant_id]/emms2//SetResp/21881E000183/SubTelecontrol
上行 payload:
```json
{
  "funcId":"SubTelecontrol",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```

----

### 云平台下发告警使能规则
下发数组内哪个ID出现,禁用哪个,清空后全部允许

下行 topic:    third/[tenant_id]/emms2//Set/21881E000183/AlarmDisable
下行 payload:

```json
{
  "funcId":"AlarmDisable",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "messages":
   {
        "alarmCodes":[70001,70002]
   }     
}
```

上行 topic:    third/[tenant_id]/emms2//SetResp/21881E000183/AlarmDisable
上行 payload:
```json
{
  "funcId":"AlarmDisable",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```

----

### 控制器&南向设备报警上报
变化上报, 哪个变化上报哪个点位,EMS和南向设备复用
上行 topic:   third/[tenant_id]/emms2//LcPost/21881E000183/Alarm
上行 payload:
```json
{
  "funcId":"Alarm",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "messages":[
   {
        "alarmCode":70001,
        "reasonCode":2,
        "severity":0,
        "entityInstance": "[Sub_ID]", //发生告警实例
        "additionalParam":"0,0",
        "raiseTime":1662002513 // 告警发生时间
   }
  ]      
}
```

> 对于不同的设备很难搞,要设备型号转

### 控制器&南向设备报警同步
下行 topic:    third/[tenant_id]/emms2//Get/21881E000183/AllAlarm
下行 payload:
```json
{
  "funcId":"AllAlarm",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
}  
```
### 全量回复告警的点位
上行 topic:   third/[tenant_id]/emms2//GetResp/21881E000183/AllAlarm
上行 payload:
```json
{
  "funcId":"AllAlarm",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "messages":[
   {
        "alarmCode":70001,
        "reasonCode":2,
        "severity":0,
        "entityInstance": "[Sub_ID]", //发生告警实例
        "additionalParam":"0,0",
        "raiseTime":1662002513 // 告警发生时间
   },
   {
        "alarmCode":70002,
        "reasonCode":2,
        "severity":0,
        "entityInstance": "[Sub_ID]", //发生告警实例
        "additionalParam":"0,0",
        "raiseTime":1662002513 // 告警发生时间
   }
  ]      
}
```

----
### 大屏数据推送控制
控制器收到命令后开启/关闭快速数据上报,超时时间后自动停止
下行 topic:    third/[tenant_id]/emms2//Set/21881E000183/DashboardCtrl
下行 payload:
```json
{
  "funcId":"DashboardCtrl",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "info":{
    "timeout":60, //默认60S后开始停止发送,可以不传
    "period":5, //默认5S发送一次,可以不传
    "action":1, //1:开启,0:停止    
  }
}  
```
上行 topic:    third/[tenant_id]/emms2//SetResp/21881E000183/DashboardCtrl
上行 payload:
```json
{
  "funcId":"DashboardCtrl",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
  "result": 0
}
```

### 大屏数据推送数据
控制器收到开启命令,按照预设周期开始快速数据上报
下行 topic:    third/[tenant_id]/emms2//LcPost/21881E000183/DashboardData
下行 payload:
```json
{
  "funcId":"DashboardData",
  "lcSN": "21881E000183",
  "seq": 150,
  "time":1662002513,
   "messages":[
    {
        "no": "EMS", //EMS状态
        "tags": {
            "ControlMode":0,        
            "StratagyType":0,     
            "SysStatus":0,        
        },                
    },
    {
        "no": "GRID_METER", //电表METER
        "tags": {
            "Online":0,                //"在线状态"
            "AlarmCnt":0,                //"告警计数"
            "AphaseVoltage":0,         //"A相电压"      
            "BphaseVoltage":0,         //"B相电压"      
            "CphaseVoltage":0,         //"C相电压"      
            "AphaseCurrent":0,         //"A相电流"      
            "BphaseCurrent":0,         //"B相电流"     
            "CphaseCurrent":0,         //"C相电流"
            "ActivePower":0,           //"瞬时总有功功率"
            "ReactivePower":0,         //"瞬时总无功功率"
        },                
    },
    {
        "no": "PCS_METER", //
        "tags": {
            "Online":0,                //"在线状态"
            "AlarmCnt":0,                //"告警计数"
            "AphaseVoltage":0,         //"A相电压"      
            "BphaseVoltage":0,         //"B相电压"      
            "CphaseVoltage":0,         //"C相电压"      
            "AphaseCurrent":0,         //"A相电流"      
            "BphaseCurrent":0,         //"B相电流"     
            "CphaseCurrent":0,         //"C相电流"
            "ActivePower":0,           //"瞬时总有功功率"
            "ReactivePower":0,         //"瞬时总无功功率"
        }
    },
        {
        "no": "PCS", //
        "tags": {
            "Online":0,                //"在线状态"
            "AlarmCnt":0,                //"告警计数"
            "AphaseVoltage":0,        //A相电压         
            "BphaseVoltage":0,        //B相电压         
            "CphaseVoltage":0,        //C相电压         
            "AphaseCurrent":0,        //A相电流         
            "BphaseCurrent":0,        //B相电流         
            "CphaseCurrent":0,        //C相电流  
            "ActivePower":0,          //有功功率          
            "ReactivePower":0,        //无功功率                
            "RunStatu":0,             //运行状态 
            "GridConnected":0,             //并网状态  
        }
    },
        {
        "no": "BMS", //
        "tags": {
            "Online":0,                //"在线状态"
            "AlarmCnt":0,                //"告警计数"
            "SOC":0,                      //RackSOC                    
            "RackVoltage":0,               //Rack电池总电压 
            "RackCurrent":0,               //Rack电池总电流 
        }
    }
   ]
}  
```
> no: 设备编号 PCS_METER,与设备列表内容的子集
> tags: 和南向设备数据的子集





## 附录

### result代码定义
| 代码 |   定义   | 说明 | 备注 |
| :--: | :------: | :--: | :--: |
|  0   |   成功   |      |      |
|  1   | 无效格式 |      |      |
|      |          |      |      |

### 告警ID
告警点位ID需要在云平台预定义,告警点位样例按照 <EMS告警定义模板-导入.xlsx> 

## 问题





