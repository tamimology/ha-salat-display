# Home Assistant Prayers Times Display Screen

#### **NOTE:** *WHEN WRITING THIS GUIDE, IT WAS BASED TO FIT A* '**LENOVO TAB3 7 ESSENTIAL**'. *ALL SETTINGS AND SIZES CAN BE ADJUSTED TO FIT YOUR TABLET ACCORDINGLY.*


# Table of Contents

***First, we start by creating the needed sensors in Home Assistant***

1. <a href="#times-sensors">Times Sensors</a>
2. <a href="#prayers-sensors">Prayers Sensors</a>
3. <a href="#daylight-saving-sensor">Daylight Saving Sensor</a>
4. <a href="#hijri-date-sensors">Hijri Date Sensors</a>
5. <a href="#next-prayer-eta-sensor">Next Prayer ETA Sensor</a>


***Second, we do the Lovelace cards designs***

6. <a href="#installing-the-required-cards-integration">Installing the Required Cards Integration</a>
7. <a href="#finalising-the-design">Finalising the Design</a>


***Finally, we test our setup***

8. <a href="#viewing-the-final-dashboard-result">Viewing the Final Dashboard Result</a>
9. <a href="#setting-up-the-tablet">Setting Up The Tablet</a>


## Times Sensors

Add the below times sensors in the `configuration.yaml` file

```
sensor:
# Time Seonsors - Parameters available at https://strftime.org/
  - platform: time_date
    display_options:
      - "time"
      - "date"
      - "date_time"
      - "date_time_utc"
      - "date_time_iso"
      - "time_date"
      - "time_utc"
      - "beat"
```

## Prayers Sensors

In your Home Assistant, goto ***Settings***, then ***Devices & services***, and on the right bottom corner, click on ***+ ADD INTEGRATION***

![prayers_integration](/screenshot/prayers_integration.png)


Search for the **Islamic Prayer Times** and enter the required parameters, as in below

![prayers_parameters](/screenshot/prayers_parameters.png)


## Daylight Saving Sensor
##### The below is specifically for Melbourne/Australia, adjust according to your timezone

In your `configuration.yaml`, add the below binary sensor to check for DST

```
template: 
# DST Binary Sensor
  #begins at 2am on the first Sunday in October
  #ends at 2am on the first Sunday in April
  - trigger:
    - platform: time
      at: '02:10'
    - platform: homeassistant
      event: start
    binary_sensor:
      - name: "dst"
        state: >
            {% if now().isoweekday() == 7 and now().hour >= 2 and now().month == 10 and now().month != (now() - timedelta(days=7)).month %} 
              {{ "on" }}
            {% elif now().day in (1,2,3,4,5,6,7) and now().hour >= 2 and now().month == 10 and now().month != (now() - timedelta(days=7)).month %}
              {{ "on" }}
            {% elif now().hour >= 2 and now().month == 10 and now().month != (now() - timedelta(days=7)).month %}
              {{ "off" }}
            {% elif now().month in (10,11,12,1,2,3) %}
              {{ "on" }}

            {% elif now().isoweekday() == 7 and now().hour >= 2 and now().month == 4 and now().month != (now() - timedelta(days=7)).month %}
              {{ "off" }}
            {% elif now().day in (1,2,3,4,5,6,7) and now().hour >= 2 and now().month == 4 and now().month != (now() - timedelta(days=7)).month %}
              {{ "off" }}
            {% elif now().hour >= 2 and now().month == 4 and now().month != (now() - timedelta(days=7)).month %}
              {{ "on" }}
            {% elif now().month in (4,5,6,7,8,9) %}
              {{ "off" }}
            {% endif %}        
        attributes: 
          value: >
              {{ "0" }}
        icon: >
          mdi:timelapse
```

## Hijri Date Sensors
##### The below sensor will convert today's Gregorian date to Hijri date using the RESTful API

In your `configuration.yaml`, add the below sensors to get the Hijri Date, Day, Arabic Weekday, Arabic Month and Hijra Year

```
rest: 
# Gregorian to Hijri Date Converter
  - resource_template:  http://api.aladhan.com/v1/gToH/{{now().timestamp() | timestamp_custom('%d-%m-%Y')}}
    scan_interval: 3600
    sensor:
      - name: "Hijri Date"
        value_template: "{{value_json.data.hijri.date}}"
      - name: "Hijri Day"
        value_template: "{{value_json.data.hijri.day}}"
      - name: "Hijri Weekday"
        value_template: "{{value_json.data.hijri.weekday.ar}}"
      - name: "Hijri Month"
        value_template: "{{value_json.data.hijri.month.ar}}"
      - name: "Hijri Year"
        value_template: "{{value_json.data.hijri.year}}"
```

## Next Prayer ETA Sensor

In your `configuration.yaml`, add the below sensor to check for the Next Prayer ETA time
##### This part to be added after the Daylight Saving Sensor, and ommit the `template:` line from it before adding it.

```
template:
# Next Prayer ETA
  - trigger:
    - platform: time_pattern
      minutes: "/1"
    - platform: homeassistant
      event: start
    sensor:
      - name: "Next Prayer ETA"
        state: >
          {% set ns = namespace(next = [])%}
          {% set times = expand('sensor.islamic_prayer_times_fajr_prayer', 'sensor.islamic_prayer_times_dhuhr_prayer', 'sensor.islamic_prayer_times_asr_prayer', 'sensor.islamic_prayer_times_maghrib_prayer', 'sensor.islamic_prayer_times_isha_prayer') | sort(attribute = 'state',reverse = false) | map(attribute = 'state') | list %}
          {% for t in times %}
            {% if (t | as_datetime | as_local >= now()) %}
              {% set ns.next = ns.next + [t] %}
            {% endif %}
          {% endfor %}
           {{ (ns.next[0] | as_timestamp - now().timestamp()) | timestamp_custom('%-H hours %M minutes', local=false) }}
        icon: mdi:camera-timer
```

## Installing the Required Cards Integration

#### This step is REQUIRED in order to have the display as seen in the screenshot at the end of this guide

The below front-end integrations were used, they can be installing from HACS.
- [WallPanel](https://github.com/j-a-n/lovelace-wallpanel)
- [Layout Card](https://github.com/thomasloven/lovelace-layout-card)
- [Vertical Stack in Card](https://github.com/ofekashery/vertical-stack-in-card)
- [Multiple Entity Row](https://github.com/benct/lovelace-multiple-entity-row)
- [Lovelace Clock Card](https://github.com/Villhellm/lovelace-clock-card)
- [Simple Clock Card](https://github.com/fufar/simple-clock-card)
- [card-mod](https://github.com/thomasloven/lovelace-card-mod)
- [Template Entity Row](https://github.com/thomasloven/lovelace-template-entity-row)
- [Google Dark Theme](https://github.com/JuanMTech/google_dark_theme)

## Finalising the Design

#### It is advisable to have a seperate user for this display, and make sure that it is not an admin, and can only be accessed through local network only.
#### Also, I advise to have a seperate dashboard (name it '***prayers-dashboard***') and add a view in this dashboard (name it '***salat-times***') for this display which is visible only to the admin and the display user created. Make sure you choose the View type as `Horizontal (layout-card)` and add the below lines in the code area

```
width: 700
max_cols: 2
```

![view_config](/screenshot/view_config.png)

Once you created the user with the recommeneded permissions, and had the dashboard visible to that user, enter the dashboards edit mode, add click on ***+ ADD CARD*** from the bottom right corner. Choose any card as there is no difference in fact. Now click on ***SHOW CODE EDITOR*** at the lower left corner. Delete whatever is there, and add the following

```s
type: custom:vertical-stack-in-card
cards:
  - type: entities
    entities:
      - entity: binary_sensor.dst
        type: custom:multiple-entity-row
        name: توقيت صيفي
        secondary_info: Day Light Saving
        icon: ' '
        show_state: false
        card_mod:
          style: |
            :host {
              {% if states("binary_sensor.dst")  == "on" %}
                --card-mod-icon-color: green;
                color: yellow;
              {% else %}
                --card-mod-icon-color: gray;
                color: gray;
              {% endif %}
    card_mod:
      style: |
        ha-card {
          font-size: 20px;
          text-align: center;
        }
  - type: custom:simple-clock-card
    time_zone: Australia/Melbourne
    use_military: true
    hide_seconds: false
    bold_clock: false
    font_size: 3rem
    paddingLeft_size: 32px
    paddingRight_size: 32px
    paddingTop_size: 32px
    paddingBottom_size: 32px
  - type: custom:clock-card
    size: 250
    font_size: 20
    disable_seconds: false
    caption: null
    display_date: DDD DD MMM, YYYY
    theme:
      background: black
      hands: orange
      numbers: white
      border: grey
    wp_style:
      margin-top: 10px
      width: 400px
      grid-row: 1
      grid-column: 1
  - type: entities
    entities:
      - type: custom:template-entity-row
        name: |
          {{states.sensor.hijri_weekday.state}}
          {{states.sensor.hijri_day.state}}
          {{states.sensor.hijri_month.state}} 
          {{states.sensor.hijri_year.state}}
          هـ
        state: ''
    show_header_toggle: false
    state_color: false
    card_mod:
      style: |
        ha-card {
          font-size: 20px;
          text-align: center;
        }
```

Repeate the above steps, but this time, add the following into the newly created card
```
type: custom:vertical-stack-in-card
cards:
  - square: false
    columns: 1
    type: grid
    cards:
      - type: entities
        entities:
          - entity: sensor.islamic_prayer_times_fajr_prayer
            type: custom:multiple-entity-row
            name: الفجر
            secondary_info: Fajr
            format: time
            styles:
              width: 200px
              text-align: left
            tap_action:
              action: none
            hold_action:
              action: none
            double_tap_action:
              action: none
            card_mod:
              style: |
                :host {
                  {% if states("sensor.islamic_prayer_times_fajr_prayer")| as_timestamp > now().timestamp() %}
                    --card-mod-icon-color: green;
                    color: green;
                  {% endif %}
          - type: section
          - entity: sensor.islamic_prayer_times_sunrise_time
            type: custom:multiple-entity-row
            name: الشروق
            secondary_info: Sunrise
            format: time
            styles:
              width: 200px
              text-align: left
            tap_action:
              action: none
            hold_action:
              action: none
            double_tap_action:
              action: none
            card_mod:
              style: |
                :host {
                  {% if states("sensor.islamic_prayer_times_sunrise_time")| as_timestamp > now().timestamp() and now().timestamp() > states("sensor.islamic_prayer_times_fajr_prayer")| as_timestamp %}
                    --card-mod-icon-color: green;
                    color: green;
                  {% endif %}
          - type: section
          - type: conditional
            conditions:
              - condition: state
                entity: sensor.hijri_weekday
                state_not: الجمعة
            row:
              entity: sensor.dhuhr_dst
              type: custom:multiple-entity-row
              name: الظهر
              secondary_info: Dhuhur
              format: time
              styles:
                width: 200px
                text-align: left
              card_mod:
                style: |
                  :host {
                    {% if states("sensor.dhuhr_dst")| as_timestamp > now().timestamp() and now().timestamp() >  states("sensor.sunrise_dst")| as_timestamp %}
                      --card-mod-icon-color: green;
                      color: green;
                    {% endif %}
          - type: conditional
            conditions:
              - condition: state
                entity: sensor.hijri_weekday
                state: الجمعة
            row:
              entity: sensor.dhuhr_dst
              type: custom:multiple-entity-row
              name: الجمعة
              secondary_info: Juma'a
              format: time
              styles:
                width: 200px
                text-align: left
              card_mod:
                style: |
                  :host {
                    {% if states("sensor.dhuhr_dst")| as_timestamp > now().timestamp() and now().timestamp() >  states("sensor.sunrise_dst")| as_timestamp %}
                      --card-mod-icon-color: green;
                      color: green;
                    {% endif %}
          - type: section
          - entity: sensor.islamic_prayer_times_asr_prayer
            type: custom:multiple-entity-row
            name: العصر
            secondary_info: Asr
            format: time
            styles:
              width: 200px
              text-align: left
            tap_action:
              action: none
            hold_action:
              action: none
            double_tap_action:
              action: none
            card_mod:
              style: |
                :host {
                  {% if states("sensor.islamic_prayer_times_asr_prayer")| as_timestamp > now().timestamp() and now().timestamp() > states("sensor.islamic_prayer_times_dhuhr_prayer")| as_timestamp  %}
                    --card-mod-icon-color: green;
                    color: green;
                  {% endif %}
          - type: section
          - entity: sensor.islamic_prayer_times_maghrib_prayer
            type: custom:multiple-entity-row
            name: المغرب
            secondary_info: Maghrib
            format: time
            styles:
              width: 200px
              text-align: left
            tap_action:
              action: none
            hold_action:
              action: none
            double_tap_action:
              action: none
            card_mod:
              style: |
                :host {
                  {% if states("sensor.islamic_prayer_times_maghrib_prayer")| as_timestamp > now().timestamp() and now().timestamp() > states("sensor.islamic_prayer_times_asr_prayer")| as_timestamp %}
                    --card-mod-icon-color: green;
                    color: green;
                  {% endif %}
          - type: section
          - entity: sensor.islamic_prayer_times_isha_prayer
            type: custom:multiple-entity-row
            name: العشاء
            secondary_info: Isha
            format: time
            styles:
              width: 200px
              text-align: left
            tap_action:
              action: none
            hold_action:
              action: none
            double_tap_action:
              action: none
            card_mod:
              style: |
                :host {
                  {% if states("sensor.islamic_prayer_times_isha_prayer")| as_timestamp > now().timestamp() and now().timestamp() > states("sensor.islamic_prayer_times_maghrib_prayer")| as_timestamp %}
                    --card-mod-icon-color: green;
                    color: green;
                  {% endif %}
          - type: section
          - entity: sensor.islamic_prayer_times_midnight_time
            type: custom:multiple-entity-row
            name: منتصف الليل
            secondary_info: Midnight
            format: time
            styles:
              width: 200px
              text-align: left
            tap_action:
              action: none
            hold_action:
              action: none
            double_tap_action:
              action: none
            card_mod:
              style: |
                :host {
                  {% if states("sensor.islamic_prayer_times_midnight_time")| as_timestamp > now().timestamp() and now().timestamp() > states("sensor.islamic_prayer_times_isha_prayer")| as_timestamp %}
                    --card-mod-icon-color: green;
                    color: green;
                  {% endif %}
          - type: section
        card_mod:
          style: |
            ha-card {
              font-size: 180% 
            }
  - type: entities
    entities:
      - type: custom:template-entity-row
        name: الزمن المتبقي للصلاة
        secondary: Next Prayer ETA
        state: |
          {% if states("sensor.next_prayer_eta")=="unavailable" %}
            TBC
          {% else %}
            {{states("sensor.next_prayer_eta")}}
          {% endif %}
    state_color: false
    tap_action:
      action: none
    hold_action:
      action: none
    double_tap_action:
      action: none
    card_mod:
      style: |
        ha-card {
          text-align: left;
          font-size: 20px;
          color: orange;
        }
```

Now, the last piece of code is to click on the three dots on the upper right corner, and choose ***{} Raw configuuration editor***.

![edit_dashboard](/screenshot/edit-dashboard.png)

On top of what you see, insert the below lines

```
wallpanel:
  enabled: true
  enabled_on_tabs:
    - salat-times
  hide_toolbar: true
  hide_sidebar: true
  fullscreen: true
  idle_time: 0
```

### NOTE: If you needed to enter the edit mode after that, press F11 to exit fullscreen mode, and add `?edit=1` at the end of the Home Assistant instance url of this dashboard

```https://192.168.1.100:8123/prayers-dashboard/salat-times?edit=1```

## Viewing the Final Dashboard Result

#### This is the last step to make sure that nothing wrong is made during the setup, and have all integrations installed and working properly.

Install the Home Assistant companion application, login using the user dedicated to this dashboard view, and enter the local ip address for Home Assistant rather than the domain.

To make the view as Dark, which I personally prefer and based all text colouring accordingly, click on the User at the lower left corner, scroll down till you find ***Theme*** and choose ***Google Dark Theme***

![dark_theme](/screenshot/dark_theme.png)


Now if everything went perfect, then you should see the below view on your tablet

![final_dashboard](/screenshot/final_dashboard.jpg)

## Setting Up The Tablet

Last step is to make sure that the tablet has its screen stay-on while charging and having the Home Assistant dashboard on screen. To do so:

- In the companion app, click on the 4-dashes on the top left corner
- At the bottom, click on the "*Username*" to opne the *Profile* page
- Scroll down, and make sure that *Automatically close connection* is enabled

![profile](/screenshot/android/profile.png)

- Again, click on the 4-dashes on the top left corner and click on the *Settings* 
- Find the *Companion app* and click on it
- Scroll down and make sure that the *Screen orientation* is set to _Landscape_
- Make sure that *Keep screen on* is enabled

![companion_app](/screenshot/android/companion_app.png)

### TO MAKE SURE THAT THE TABLET DOES NOT TURN THE SCREEN ON, AS AN INTERNAL TABLED SETTINGS, YOU NEED TO FOLLOW THE BELOW. IT COULD BE NOT NEEDED ON SOME TABLET MODELS

- Open the Android *Settings* and click on the *About phone* at the bottom of the list

![companion_app](/screenshot/android/about_android.png)

- Click on *Software information* and keep on clicking on the *Build number* for around 7-10 clics
- Stop when you see that the _Developer mode_ is enabled

![software_info_android](/screenshot/android/software_info_android.png)

- Go back to the *Settings* screen, and at the bottom of the list, you will see *Developer options* below the *About phone* appered. Click on that

![settings_dev_android](/screenshot/android/settings_dev_android.png)

- Finally, scroll down and make sure that *Stay awake* is enabled

![developer_options_android](/screenshot/android/developer_options_android.png)


Now you should have your tablet in _ALWASY ON MODE_ with the prayers dashboard visible. Make sure you plug in the charger as well






## License
This document guide is licensed under the CC0 1.0 Universal license. The terms of the license are detailed in [LICENSE](/LICENSE)
