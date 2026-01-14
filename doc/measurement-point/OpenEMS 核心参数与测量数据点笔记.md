# OpenEMS 核心参数与测量数据点笔记

# 一、基础概念：测量数据点核心抽象

OpenEMS 中所有测量数据点的核心抽象是 `Channel`，是传感器采集值、设备运行参数、计算衍生值的统一载体，隶属于 **Component**（设备组件），通过 `ChannelId` 唯一索引。

## 核心抽象对应的Java类

| 抽象概念 | Java类路径 | 说明 |
|---------|------------|------|
| Channel | io.openems.edge.common.channel.Channel | 通道核心接口，定义了通道的基本属性和方法 |
| Component | io.openems.edge.common.component.OpenemsComponent | 设备组件核心接口，所有OpenEMS组件都实现此接口 |
| ChannelId | io.openems.edge.common.channel.ChannelId | 通道ID接口，用于唯一标识通道 |
| ChannelAddress | io.openems.common.types.ChannelAddress | 系统级唯一通道地址，由Component-ID和Channel-ID组成 |

## 1.1 Channel 核心分类

|类型|接口|Java类路径|用途|示例|
|---|---|---|---|---|
|只读通道|ReadChannel|io.openems.edge.common.channel.ReadChannel|采集设备测量数据，仅驱动可写入|电压、电流、功率|
|可写通道|WriteChannel|io.openems.edge.common.channel.WriteChannel|设备控制指令，支持读写|储能充放电功率限值|
|虚拟通道|VirtualChannel|io.openems.edge.common.channel.VirtualChannel|内部计算生成，无物理数据源|总负载功率|
|故障通道|FaultChannel|io.openems.edge.common.channel.FaultChannel|设备故障码/状态上报|逆变器故障代码|
## 1.2 Channel 核心属性

|属性|说明|示例|
|---|---|---|
|ChannelId|唯一标识符，全局不可重复|Ess0/Dc/Current、PvInverter1/Ac/ActivePower|
|Type|数据类型|Integer、Double、Boolean|
|Unit|物理单位|V、A、kW、%|
|AccessMode|访问模式|READ_ONLY、READ_WRITE|
|LastUpdateTime|最后更新时间戳，用于判断数据有效性|1735689600000（毫秒级）|
## 1.3 命名规范

采用分层命名规则：`ComponentId/[子路径]/Metric`，其中：

- ComponentId：设备组件唯一ID（如 Ess0、PvInverter2）

- 子路径：区分设备不同模块（如 Ac 交流侧、Dc 直流侧）

- Metric：具体测量指标（如 ActivePower 有功功率）

# 二、各模块核心参数与测量数据点

## 2.1 电网参数（Grid Meter 组件）

核心用于监测电网连接状态、能量流向和电能质量，覆盖基础电参数、电能计量、电能质量等。

### 对应Java类

| 组件类型 | Java接口/类 | 说明 |
|---------|------------|------|
| Grid Meter | io.openems.edge.meter.api.ElectricityMeter | 电网测量核心接口，定义了电网测量相关的通道和方法 |
| Grid Meter实现 | io.openems.edge.simulator.meter.grid.SimulatorGridMeter | 模拟器实现，用于测试和仿真 |
| Grid Meter实现 | io.openems.edge.solaredge.gridmeter.SolarEdgeGridMeter | SolarEdge电网表实现 |
| Grid Meter实现 | io.openems.edge.kostal.plenticore.gridmeter.KostalGridMeter | Kostal电网表实现 |

### 2.1.1 基础电参数（三相测量）

|ChannelId|描述|单位|
|---|---|---|
|GridMeter0/Ac/L1/Voltage|L1相电压|V|
|GridMeter0/Ac/L1/Current|L1相电流|A|
|GridMeter0/Ac/ActivePower|总有功功率（正值=购入，负值=售出）|kW|
|GridMeter0/Ac/ReactivePower|总无功功率|kVar|
|GridMeter0/Ac/Frequency|电网频率|Hz|
|GridMeter0/Ac/PowerFactor|总功率因数|无量纲|
### 2.1.2 电能计量参数（双向）

|ChannelId|描述|单位|
|---|---|---|
|GridMeter0/Ac/Energy/Active/Import|当日购入有功电能|kWh|
|GridMeter0/Ac/Energy/Active/Export|当日售出有功电能|kWh|
|GridMeter0/Ac/Energy/Active/ImportTotal|累计购入有功电能|kWh|
### 2.1.3 电能质量参数

|ChannelId|描述|单位|
|---|---|---|
|GridMeter0/Ac/VoltageTHD|电压总谐波畸变率|%|
|GridMeter0/Ac/CurrentUnbalance|电流不平衡度|%|
## 2.2 光伏参数（PvInverter/PvArray 组件）

覆盖直流侧发电特性、交流侧并网参数、能量计量、设备状态及环境监测，核心支撑光伏出力评估与优化。

### 对应Java类

| 组件类型 | Java接口/类 | 说明 |
|---------|------------|------|
| PvInverter | io.openems.edge.pvinverter.api.ManagedSymmetricPvInverter | 光伏逆变器核心接口，定义了光伏逆变器相关的通道和方法 |
| PvInverter实现 | io.openems.edge.simulator.pvinverter.SimulatorPvInverter | 模拟器实现，用于测试和仿真 |
| PvInverter实现 | io.openems.edge.solaredge.pvinverter.SolarEdgePvInverter | SolarEdge光伏逆变器实现 |
| PvInverter实现 | io.openems.edge.sma.pvinverter.PvInverterSmaSunnyTripower | SMA Sunny Tripower光伏逆变器实现 |
| PvInverter实现 | io.openems.edge.fronius.pvinverter.PvInverterFronius | Fronius光伏逆变器实现 |

### 2.2.1 直流侧核心参数

|ChannelId|描述|单位|
|---|---|---|
|PvInverter0/Dc/Input1/Voltage|直流输入1路电压|V|
|PvInverter0/Dc/TotalPower|直流总输入功率|W|
|PvInverter0/Dc/MaxPowerPointVoltage|MPPT电压|V|
### 2.2.2 交流侧并网参数

|ChannelId|描述|单位|
|---|---|---|
|PvInverter0/Ac/TotalActivePower|总有功功率（正值=向电网馈电）|W|
|PvInverter0/Ac/Frequency|输出频率（跟踪电网）|Hz|
|PvInverter0/Ac/VoltageTHD|电压总谐波畸变率|%|
### 2.2.3 环境监测参数（PvArray 组件）

|ChannelId|描述|单位|
|---|---|---|
|PvArray0/Irradiance|太阳辐照度|W/m²|
|PvArray0/BackSheetTemperature|光伏组件背板温度|°C|
|PvArray0/PerformanceRatio|性能比（实际/理论发电量）|%|
## 2.3 储能参数（Battery/BatteryInverter/Ess 组件）

覆盖电池状态（SOC/SOH）、充放电特性、并网参数、安全保护，是储能系统控制的核心依据。

### 对应Java类

| 组件类型 | Java接口/类 | 说明 |
|---------|------------|------|
| Battery | io.openems.edge.battery.api.Battery | 电池核心接口，定义了电池相关的通道和方法 |
| Battery实现 | io.openems.edge.simulator.battery.SimulatorBattery | 模拟器实现，用于测试和仿真 |
| Battery实现 | io.openems.edge.victron.battery.VictronBattery | Victron电池实现 |
| BatteryInverter | io.openems.edge.batteryinverter.api.ManagedSymmetricBatteryInverter | 储能逆变器核心接口 |
| BatteryInverter实现 | io.openems.edge.victron.batteryinverter.VictronBatteryInverter | Victron储能逆变器实现 |
| Ess | io.openems.edge.ess.api.SymmetricEss | 对称储能系统接口 |
| Ess | io.openems.edge.ess.api.AsymmetricEss | 非对称储能系统接口 |
| Ess实现 | io.openems.edge.simulator.ess.symmetric.SimulatorEssSymmetric | 模拟器实现，用于测试和仿真 |
| Ess实现 | io.openems.edge.fenecon.mini.ess.FeneconMiniEss | Fenecon Mini储能系统实现 |
| Ess实现 | io.openems.edge.goodwe.ess.GoodWeEss | GoodWe储能系统实现 |

### 2.3.1 电池核心参数（Battery 组件）

|ChannelId|描述|单位|
|---|---|---|
|Battery0/Soc|荷电状态（剩余电量占比）|%|
|Battery0/Soh|健康状态（当前/额定容量比）|%|
|Battery0/Dc/Voltage|电池总电压|V|
|Battery0/Dc/Current|电池电流（正值=充电，负值=放电）|A|
|Battery0/Temperature/Average|电池平均温度|°C|
### 2.3.2 储能逆变器参数（BatteryInverter 组件）

|ChannelId|描述|单位|
|---|---|---|
|BatteryInverter0/Ac/TotalActivePower|总有功功率（正值=放电，负值=充电）|kW|
|BatteryInverter0/State/GridConnected|并网状态（true=已并网）|Boolean|
|BatteryInverter0/Ac/Energy/Active/ImportTotal|累计充电电能|kWh|
### 2.3.3 ESS 聚合组件参数

|ChannelId|描述|单位|
|---|---|---|
|Ess0/Soc|系统级SOC（与Battery0/Soc同步）|%|
|Ess0/ActivePower|系统总有功功率|kW|
|Ess0/State|系统运行状态（IDLE/CHARGING/DISCHARGING）|Enum|
## 2.4 负载参数（Load/FlexibleLoad 组件）

覆盖电气特性、能量计量、柔性控制，支持固定负载监测与柔性负载智能调度。

### 对应Java类

| 组件类型 | Java接口/类 | 说明 |
|---------|------------|------|
| Load | io.openems.edge.meter.api.ElectricityMeter | 负载监测通过电表接口实现，定义了负载测量相关的通道和方法 |
| Load实现 | io.openems.edge.meter.pqplus.umd97.MeterPqplusUmd97Impl | PQ+ UMD97电表实现，可用于负载监测 |
| Load实现 | io.openems.edge.meter.pqplus.umd96.MeterPqplusUmd96Impl | PQ+ UMD96电表实现，可用于负载监测 |
| 柔性负载控制 | io.openems.edge.controller.highloadtimeslot.

### 2.4.1 核心电气与计量参数

|ChannelId|描述|单位|
|---|---|---|
|Load0/Ac/TotalActivePower|总有功功率（正值=消耗能量）|kW|
|Load0/Ac/Energy/Active/Import|当日能耗|kWh|
|Load0/Ac/PowerFactor|功率因数|无量纲|
### 2.4.2 柔性负载控制参数

|ChannelId|描述|单位|
|---|---|---|
|FlexibleLoad0/Control/ActivePowerMax|最大有功功率限制|kW|
|FlexibleLoad0/Control/Interruptible|可中断标识（true=可中断）|Boolean|
|FlexibleLoad0/Control/ShiftableWindowStart|可平移时段起始|ISO 8601|
## 2.5 环境参数（Weather/Environment 组件）

覆盖气象、光照、电价等，支撑光储出力预测、负载优化与经济调度。

### 对应Java类

| 组件类型 | Java接口/类 | 说明 |
|---------|------------|------|
| Weather | io.openems.edge.weather.api.Weather | 气象数据核心接口，定义了气象相关的通道和方法 |
| Weather实现 | io.openems.edge.weather.openmeteo.WeatherOpenMeteoImpl | OpenMeteo气象服务实现 |
| Weather实现 | io.openems.edge.weather.test.DummyWeather | 模拟器实现，用于测试和仿真 |
| TimeOfUseTariff | io.openems.edge.timeofusetariff.api.TimeOfUseTariff | 分时电价核心接口 |
| TimeOfUseTariff实现 | io.openems.edge.timeofusetariff.tibber.TimeOfUseTariffTibberImpl | Tibber分时电价实现 |
| TimeOfUseTariff实现 | io.openems.edge.timeofusetariff.corrently.TimeOfUseTariffCorrentlyImpl | Corrently分时电价实现 |
| TimeOfUseTariff实现 | io.openems.edge.timeofusetariff.awattar.TimeOfUseTariffAwattarImpl | Awattar分时电价实现 |

### 2.5.1 核心气象参数

|ChannelId|描述|单位|
|---|---|---|
|Weather0/AirTemperature|环境温度|°C|
|Weather0/WindSpeed|风速|m/s|
|Weather0/Sun/Elevation|太阳高度角|°|
### 2.5.2 光伏相关环境参数

|ChannelId|描述|单位|
|---|---|---|
|Environment0/Irradiation/PlaneOfArray|光伏阵列面辐照|W/m²|
|Environment0/Irradiation/GlobalHorizontal|水平面总辐照|W/m²|
### 2.5.3 电价环境参数

|ChannelId|描述|单位|
|---|---|---|
|TimeOfUseTariff0/ElectricityPrice/Buy|当前购电价|元/kWh|
|TimeOfUseTariff0/ElectricityPrice/PeakHour|峰时标识（true=高峰时段）|Boolean|
## 2.6 设备状态参数（所有组件通用）

覆盖运行状态、连接状态、健康状态、故障状态，是系统监控与故障自愈的核心依据。

### 对应Java类

| 组件类型 | Java接口/类 | 说明 |
|---------|------------|------|
| OpenemsComponent | io.openems.edge.common.component.OpenemsComponent | 所有OpenEMS组件的核心接口，包含设备状态相关的通道 |
| Channel | io.openems.edge.common.channel.Channel | 通道接口，用于存储和传递设备状态信息 |
| StateCollectorChannel | io.openems.edge.common.channel.internal.StateCollectorChannel | 状态收集通道，用于汇总设备的各种状态 |
| Level | io.openems.common.channel.Level | 定义了设备状态的级别（OK/INFO/WARNING/ERROR/CRITICAL） |

### 2.6.1 通用运行与连接状态

|ChannelId|描述|数据类型|
|---|---|---|
|ComponentId/State/Running|运行状态（true=运行中）|Boolean|
|ComponentId/State/Enabled|启用状态（true=已启用）|Boolean|
|ComponentId/State/Connected|通信状态（true=通信正常）|Boolean|
|ComponentId/State/GridConnected|并网状态（true=已并网）|Boolean|
### 2.6.2 故障与保护状态

|ChannelId|描述|数据类型|
|---|---|---|
|ComponentId/State/FaultActive|故障存在标识（true=有故障）|Boolean|
|ComponentId/State/FaultCode|故障代码|Integer|
|ComponentId/State/FaultLevel|故障等级（INFO/WARNING/ERROR/CRITICAL）|Enum|
|ComponentId/State/OverTemperatureProtection|过热保护状态（true=已触发）|Boolean|
# 三、关键代码使用示例

## 3.1 获取设备实时参数

```java

import io.openems.edge.common.component.ComponentManager;
import io.openems.edge.ess.api.SymmetricEss;

public class EssMonitor {
    private final ComponentManager componentManager;

    // 注入ComponentManager（OSGi依赖注入）
    public EssMonitor(ComponentManager componentManager) {
        this.componentManager = componentManager;
    }

    // 获取储能系统核心状态
    public EssState getEssRealTimeState() {
        EssState state = new EssState();
        SymmetricEss ess = componentManager.getComponent("Ess0");
        if (ess == null) return state;

        // 获取SOC
        state.soc = ess.getSocChannel()
                .value()
                .orElse(0.0);

        // 获取总有功功率
        state.activePower = ess.getActivePowerChannel()
                .value()
                .orElse(0.0);

        // 获取运行状态
        state.isRunning = ess.isEnabled();

        // 获取故障状态
        state.hasFault = ess.hasFaults();

        return state;
    }

    // 数据封装类
    public static class EssState {
        public double soc;          // 荷电状态(%)
        public double activePower;  // 总有功功率(kW)
        public boolean isRunning;   // 运行状态
        public boolean hasFault;    // 故障状态
    }
}
```

## 3.2 柔性负载控制

```java

import io.openems.edge.common.component.ComponentManager;
import io.openems.edge.meter.api.ElectricityMeter;

public class FlexibleLoadController {
    private final ComponentManager componentManager;

    public FlexibleLoadController(ComponentManager componentManager) {
        this.componentManager = componentManager;
    }

    // 调整柔性负载功率
    public void adjustFlexibleLoadPower(double targetPower) {
        ElectricityMeter flexibleLoad = componentManager.getComponent("FlexibleLoad0");
        if (flexibleLoad == null) return;

        // 此处仅为示例，实际柔性负载控制需要根据具体实现调整
        // 不同的负载设备可能有不同的控制通道和方法
        
        System.out.printf("柔性负载目标功率：%.2fkW%n", targetPower);
    }
}
```

# 四、配置与使用要点

1. **命名规范**：建议使用标准组件命名（如 Ess0、PvInverter0、Load0），便于系统集成与维护。

2. **数据采集频率**：实时参数（功率、状态）建议 1 秒/次；非实时参数（SOC、能耗）建议 1 分钟/次；健康参数（SOH）建议 5-15 分钟/次。

3. **功率方向规则**：
        

    - 光伏逆变器：有功功率正值=向电网馈电

    - 储能系统：有功功率正值=放电，负值=充电

    - 负载：有功功率始终为正值（消耗能量）

4. **故障处理**：实时监测 FaultActive 与 FaultCode，触发故障时自动切换设备至安全模式（停机/降额）。

5. **数据有效性**：通过 LastUpdateTime 判断数据是否超时，超时数据标记为无效，避免错误调度。
> （注：文档部分内容可能由 AI 生成）