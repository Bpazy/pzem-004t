# pzem-004t
培正电表用电监测，基于 ESPhome，使用 PZEM-004t, DHT20 和 ESP32 WROOM 将数据集成到 HomeAssistant

硬件详情跳转: https://github.com/Bpazy/blog/issues/330

## Home Assistant 配置
为了配置「能源」模块，还需要在 `configuration.yaml` 中新增以下内容:
```yaml
sensor:
  # 实时电价传感器
  - platform: template
    sensors:
      shishidianjia:
        unit_of_measurement: "/kWh"
        friendly_name:  '实时电价'
        value_template: >
          {% if now().strftime("%H")| int >= 8 and now().strftime("%H")|int < 21 and states("sensor.current_stage")=="1" %}
            0.5583
          {% elif now().strftime("%H")| int >= 8 and now().strftime("%H")|int < 21  and states("sensor.current_stage")=="2" %}
            0.6083
          {% elif now().strftime("%H")| int >= 8 and now().strftime("%H")|int < 21  and states("sensor.current_stage")=="3" %}
            0.8583
          {% elif states("sensor.current_stage")=="1"%}
            0.3583
          {% elif states("sensor.current_stage")=="2" %}
            0.4083
          {% elif states("sensor.current_stage")=="3" %}
            0.6583
          {% endif %}         

  # 阶梯电价传感器
  - platform: template
    sensors:
       current_stage:
         value_template: >
           {% if states("sensor.pzem004_yearly_energy") | float <= 2760000 %}
             1
           {% elif states("sensor.pzem004_yearly_energy") | float > 2760000 and states("sensor.pzem004_yearly_energy") | float <= 4800000 %}
             2
           {% elif states("sensor.pzem004_yearly_energy") | float > 4800000 %}
             3
           {% endif %}
         friendly_name:  '当前阶梯'
         unit_of_measurement: "L"

# 基于 pzem004 energy 数据创建月维度统计、年维度统计传感器
utility_meter:
  pzem004_monthly_energy:
    source: sensor.pzem004_energy
    name: Pzem004 Monthly Energy
    cycle: monthly
  pzem004_yearly_energy:
    source: sensor.pzem004_energy
    name: Pzem004 Yearly Energy
    cycle: yearly

```

**这样就可以做下图这样的选择了:**

<img width="200" alt="image" src="https://github.com/user-attachments/assets/d24fb3fd-8b1f-470f-bc91-f6bbf1c1fe24">

**效果如图:**

<img width="925" alt="image" src="https://github.com/user-attachments/assets/4e9c475f-6c55-4c1d-85bf-2744138cd541">

