<p>#1 My home smart heating configuration with the use of Home Assistant</p>
<p>Home Assistant is Open Source software that runs on various devices, for instance Raspberry Pi, and acts as a central server for smart home devices and/or self build modules to make automatizations in the home. It has an active community and a large library of integrations with products on the market. Home Assistant is a non-cloud system, which means there is not necessarily a dependance on external cloud services (and an internet connection).</p>
<figure>
<img src="https://user-images.githubusercontent.com/43075793/110300847-b4172f00-7ff7-11eb-94d2-3394992bd090.png" alt="https://www.home-assistant.io/" /><figcaption>https://www.home-assistant.io/</figcaption>
</figure>
<p>With the use of Home Assistant I created a smart thermostat, which I was able to customize to my personal situation. Against low costs and all wired to limit the EMF radiation in the home. In this document I will describe my configuration. I am Dutch, so the names I chose for various entities are in Dutch.</p>
<p>Screenshots of my dashboard:</p>
<p><img src="https://user-images.githubusercontent.com/43075793/110309572-ce560a80-8001-11eb-85fc-a04b34ca5de8.png" /></p>
<p>Here is a tab with variables:</p>
<p><img src="https://user-images.githubusercontent.com/43075793/110309259-699ab000-8001-11eb-95a1-4a28d13302c3.png" /></p>
<p>And on my phone:</p>
<p><img src="https://user-images.githubusercontent.com/43075793/110310007-51776080-8002-11eb-9b64-755373b3a415.png" /></p>
<ul>
<li>TOC<br />
{:toc}</li>
</ul>
<p>My appartement consists of a living room, a bedroom and a kitchen.</p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/117958666-8a41f780-b31b-11eb-812e-aa2912945f4b.png">
</figure>
</td>
</tr>
<tr>
<td>
A schematic view of the floorplan with the relevant rooms
</td>
</tr>
</tbody>
</table>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/110297333-a790d780-7ff3-11eb-839d-b3b001fa90a7.png">
</figure>
</td>
</tr>
<tr>
<td>
A 3D view of the floorplan, the door between living room and bedroom can be closed off<br>(made with <a href="https://roomstyler.com/3dplanner">https://roomstyler.com/3dplanner</a>)
</td>
</tr>
</tbody>
</table>
<p>The appartement has a boiler (Intergas kompakt hre 24/18) for central heating which can use a on-off control and OpenTherm.  </p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/117959173-08060300-b31c-11eb-9171-167f414ecc1a.png">
</figure>
</td>
</tr>
<tr>
<td>
Intergas hre 24/18
</td>
</tr>
</tbody>
</table>
<p>##1 The design wishes for the system</p>
<ul>
<li>Multi zone: Heat the bedroom to a desired temperature at night (while living room radiator closed) and the living room during the day and evening (while bedroom radiator closed)</li>
<li>Turn off the thermostat function when no one home</li>
<li>A wired system, that doesn’t use wifi, to limit the use of EMF-radiation in the home. See <a href="https://www.youtube.com/watch?v=F0NEaPTu9oI">this TED talk</a> about possible dangers of EMF radiation in the living environment.</li>
<li>A smartphone app and / or a browser to control and monitor the thermostat</li>
<li>Send alerts to the smartphone when certain conditions are met, e.g. when it is suspected that a window is open.</li>
<li>Reverting to normal set temperature after a certain amount of time after a manual change</li>
<li>Easily setting variables like bedtime, waking time and revert-to-initial-time with input fields in the front-end</li>
</ul>
<h2 id="software"><span class="header-section-number">0.1</span> Software</h2>
<p><a href="https://www.home-assistant.io/">Home Assistant</a> (The main platform on which everything is running)</p>
<p>Arduino IDE (for uploading Arduino sketches to the arduino mcu)</p>
<p><strong>Home assistant integrations for the thermostat</strong></p>
<p>Several integrations are used. The most important is the Generic Thermostat integration, designed to act as a on-off thermostat with a switch:</p>
<ul>
<li><a href="https://www.home-assistant.io/integrations/generic_thermostat/">Generic Thermostat integration</a></li>
</ul>
<h2 id="hardware"><span class="header-section-number">0.2</span> Hardware</h2>
<p><strong>Bedroom</strong></p>
<ul>
<li>Arduino nano with a DS18B20 temperature sensor connected via USB to the Home Assistant server</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/121677790-abcb0600-cab6-11eb-9732-bff4973d8bb1.png" /></p>
<p><img src="https://user-images.githubusercontent.com/43075793/117957906-cb85d780-b31a-11eb-8d61-c71f36264ce6.png" /></p>
<p>Store: <a href="https://nl.aliexpress.com/item/1005002674336082.html">Aliexpress</a></p>
<p>Arduino sketch used:</p>
<div class="sourceCode" id="cb1"><pre class="sourceCode cpp"><code class="sourceCode cpp"><a class="sourceLine" id="cb1-1" data-line-number="1"><span class="pp">#include </span><span class="im">&lt;OneWire.h&gt;</span></a>
<a class="sourceLine" id="cb1-2" data-line-number="2"><span class="pp">#include </span><span class="im">&lt;DallasTemperature.h&gt;</span></a>
<a class="sourceLine" id="cb1-3" data-line-number="3"></a>
<a class="sourceLine" id="cb1-4" data-line-number="4"><span class="co">// Data wire is plugged into port 2 on the Arduino</span></a>
<a class="sourceLine" id="cb1-5" data-line-number="5"><span class="pp">#define ONE_WIRE_BUS 2</span></a>
<a class="sourceLine" id="cb1-6" data-line-number="6"></a>
<a class="sourceLine" id="cb1-7" data-line-number="7"><span class="co">// Setup a oneWire instance to communicate with any OneWire devices (not just Maxim/Dallas temperature ICs)</span></a>
<a class="sourceLine" id="cb1-8" data-line-number="8">OneWire oneWire(ONE_WIRE_BUS);</a>
<a class="sourceLine" id="cb1-9" data-line-number="9"></a>
<a class="sourceLine" id="cb1-10" data-line-number="10"><span class="co">// Pass our oneWire reference to Dallas Temperature.</span></a>
<a class="sourceLine" id="cb1-11" data-line-number="11">DallasTemperature sensors(&amp;oneWire);</a>
<a class="sourceLine" id="cb1-12" data-line-number="12"></a>
<a class="sourceLine" id="cb1-13" data-line-number="13"><span class="co">/*</span></a>
<a class="sourceLine" id="cb1-14" data-line-number="14"><span class="co">   The setup function. We only start the sensors here</span></a>
<a class="sourceLine" id="cb1-15" data-line-number="15"><span class="co">*/</span></a>
<a class="sourceLine" id="cb1-16" data-line-number="16"><span class="dt">void</span> setup(<span class="dt">void</span>)</a>
<a class="sourceLine" id="cb1-17" data-line-number="17">{</a>
<a class="sourceLine" id="cb1-18" data-line-number="18">  <span class="co">// start serial port</span></a>
<a class="sourceLine" id="cb1-19" data-line-number="19">  Serial.begin(<span class="dv">9600</span>);</a>
<a class="sourceLine" id="cb1-20" data-line-number="20"></a>
<a class="sourceLine" id="cb1-21" data-line-number="21">  <span class="co">// Start up the library</span></a>
<a class="sourceLine" id="cb1-22" data-line-number="22">  sensors.begin();</a>
<a class="sourceLine" id="cb1-23" data-line-number="23">  sensors.setResolution(<span class="dv">12</span>);</a>
<a class="sourceLine" id="cb1-24" data-line-number="24">}</a>
<a class="sourceLine" id="cb1-25" data-line-number="25"></a>
<a class="sourceLine" id="cb1-26" data-line-number="26"><span class="co">/*</span></a>
<a class="sourceLine" id="cb1-27" data-line-number="27"><span class="co">   Main function, get and show the temperature</span></a>
<a class="sourceLine" id="cb1-28" data-line-number="28"><span class="co">*/</span></a>
<a class="sourceLine" id="cb1-29" data-line-number="29"><span class="dt">void</span> loop(<span class="dt">void</span>)</a>
<a class="sourceLine" id="cb1-30" data-line-number="30">{</a>
<a class="sourceLine" id="cb1-31" data-line-number="31">  <span class="dt">int</span> numberOfDevices = sensors.getDeviceCount();</a>
<a class="sourceLine" id="cb1-32" data-line-number="32">  <span class="cf">if</span> (numberOfDevices &gt; <span class="dv">0</span>) {</a>
<a class="sourceLine" id="cb1-33" data-line-number="33">    sensors.requestTemperatures();</a>
<a class="sourceLine" id="cb1-34" data-line-number="34">      }</a>
<a class="sourceLine" id="cb1-35" data-line-number="35">      <span class="dt">float</span> t = sensors.getTempCByIndex(<span class="dv">0</span>);</a>
<a class="sourceLine" id="cb1-36" data-line-number="36">      Serial.println(t);</a>
<a class="sourceLine" id="cb1-37" data-line-number="37"></a>
<a class="sourceLine" id="cb1-38" data-line-number="38">  delay(<span class="dv">30000</span>);</a>
<a class="sourceLine" id="cb1-39" data-line-number="39">}</a></code></pre></div>
<p><strong>Living room</strong></p>
<p><strong>Server with sensors:</strong></p>
<p><img src="https://user-images.githubusercontent.com/43075793/121671494-e6c93b80-caae-11eb-8e96-3eeadb19c349.png" /></p>
<p>Case: <a href="https://s.click.aliexpress.com/e/_ABlPWR">Aliexpress</a><br />
PIR case (only used as case and fitted in 5V PIR): <a href="https://s.click.aliexpress.com/e/_A9CbmJ">Aliexpress</a></p>
<p>Contains:</p>
<ul>
<li>Raspberry pi Zero W (acts as server) with:</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/117957217-210db480-b31a-11eb-834b-1250b6fbd008.png" /></p>
<ul>
<li>DS18B20 temperature sensor</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/117957906-cb85d780-b31a-11eb-8d61-c71f36264ce6.png" /></p>
<p>Store: <a href="https://nl.aliexpress.com/item/1005002674336082.html">Aliexpress</a></p>
<ul>
<li>PIR motion sensor module</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/117958088-f8d28580-b31a-11eb-999f-f8b17d1bf0c5.png" /></p>
<p>Store: <a href="https://s.click.aliexpress.com/e/_9HU2TH">Aliexpress</a></p>
<ul>
<li>Relay module with wire to boiler for controlling on-off heating</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/117958501-5c5cb300-b31b-11eb-8065-645693a0284e.png" /></p>
<p>Store: <a href="https://s.click.aliexpress.com/e/_A5z0ab">Aliexpress</a></p>
<ul>
<li>USB hub with Ethernet and connected USB storage device for the OS.</li>
<li>MicroSD with bootcode.bin file (needed, as pi0 doesn’t boot from USB by default)</li>
</ul>
<p>The pi Zero W is the Raspberry with the least specifications. However it does run well for a year now, as I only use it for nothing else than the thermostat function. `</p>
<p>Dowload Home-Assistant  OS for pi0 at: <a href="https://github.com/home-assistant/operating-system/releases" class="uri">https://github.com/home-assistant/operating-system/releases</a> for pi Zero W choose: hassos_rpi0-w-x.xx.img.xz. </p>
<p>Both rooms have one radiator, each is equipped with a eqiva-N thermostatic radiator valve (TRV). These are only used to be open and close the radiators at the beginning and end of the day. They are programmed to setpoint 12° C when the desired heating for the room is off and to 25° C when the desired heating is on. The limitation of using these TRVs is that they aren’t accessible from within Home Assistant, as I chose a non smart TRV.</p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/102992726-0a59f300-451c-11eb-83e4-a0b0d94b93bb.jpg">
</figure>
</td>
</tr>
<tr>
<td>
<i>The eqiva-N thermostatic radiator valve (TRV)</i>
</td>
</tr>
</tbody>
</table>
<p>Radiator valve before:</p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/102994879-41320800-4520-11eb-9485-fe36122d2637.jpg">
</figure>
</td>
</tr>
<tr>
<td>
<i>The Herz valve knob - old situation</i>
</td>
</tr>
</tbody>
</table>
<p>There were no thermostatic valves on my radiator (just a turn knob), to get it working with the Eqiva TRV I had to put in a new insert, part #1639091 for Herz radiator valves:  </p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/102867026-6d725980-4438-11eb-9752-5bd6fffe2686.png" alt="">
</figure>
</td>
</tr>
<tr>
<td>
<i>The Herz 1639091 insert to make the valve a thermostatic valve</i>
</td>
</tr>
</tbody>
</table>
<p>An extra adapter was necessary to fit the eqiva-N TRV as it doesn’t fit the Herz system with the adapters provided on buying. </p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/102991803-40967300-451a-11eb-9a69-aa1a5d3b32cc.jpg">
</figure>
</td>
</tr>
<tr>
<td>
<i>IMI adapter for Herz part no. 15071304</i>
</td>
</tr>
</tbody>
</table>
<p>A doorspring was put on both rooms door, so that they will be kept closed as much as possible to avoid heat loss.  </p>
<table>
<tbody>
<tr>
<td>
<figure class="image">
<img src="https://user-images.githubusercontent.com/43075793/102992183-0b3e5500-451b-11eb-8786-723f359d2996.jpeg">
</figure>
</td>
</tr>
<tr>
<td>
<i>Doorspring for door closing</i>
</td>
</tr>
</tbody>
</table>
<h2 id="configuration-in-home-assistant"><span class="header-section-number">0.3</span> Configuration in Home Assistant</h2>
<h3 id="helpers"><span class="header-section-number">0.3.1</span> Helpers </h3>
<p>Created the following helpers via configuration &gt; helpers within Home Assistant</p>
<table>
<thead>
<tr class="header">
<th> </th>
<th>Type</th>
<th> </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Someone home?</strong></td>
<td>input_boolean (toggle)</td>
<td> </td>
</tr>
<tr class="even">
<td> </td>
<td> </td>
<td>input_datetime.bedtijd</td>
</tr>
<tr class="odd">
<td> </td>
<td> </td>
<td>input_text.woonk_nacht</td>
</tr>
<tr class="even">
<td> </td>
<td> </td>
<td>input_number.current_insteltemp_woonkamer</td>
</tr>
</tbody>
</table>
<h3 id="variables"><span class="header-section-number">0.3.2</span> Variables</h3>
<p>To easily change settings to which I would like them, I make use of input variables. I made a tab in Lovelace to edit this variables.</p>
<p><img src="https://user-images.githubusercontent.com/43075793/121800703-a941eb00-cc33-11eb-9dcc-9041997f328b.png" /></p>
<p>Most of them speak for themselves. The someone home switch is turned on depending on presence detection. When the set temperature of the thermostat is changed manually, it will revert back to the initial set temperature according to program after the set Duration manual change value. The countdown shows the amount of time left until revert to initial set temperature. </p>
<h3 id="configuration.yaml"><span class="header-section-number">0.3.3</span> Configuration.yaml</h3>
<p>Only covering the relevant part of the configuration for the smart heating system. </p>
<h4 id="for-the-generic-thermostat-integration"><span class="header-section-number">0.3.3.1</span> For the generic thermostat integration</h4>
<p>Home Assistant documentation: <a href="https://www.home-assistant.io/integrations/generic_thermostat/">Generic thermostat</a></p>
<p>Choices for configuration variables:</p>
<ul>
<li><code>target_temp</code> I chose target temps within configuration.yaml lower than I actually want, cause with a sudden a reboot of the system I don’t want the heating to turn on. I use automations to override the temperature to set these to desired temperatures. After a restart the <code>target_temp</code> is reverted back to the last setting before the reboot. See Automations further on.</li>
<li><code>min_cycle_duration</code> Set so that pump and gas furnace of boiler don’t have to turn on and off that often. Set to 3 minutes for bedroom and 1 minute for living room. The living room has a larger radiator and 1 minute turned out as a good value. For the bedroom 3 minutes as the there is a smaller radiator.</li>
<li><code>heater</code> I use two <code>generic_thermostat</code> instances with separate heater switches, because Generic thermostat isn’t able to work with multiple zones (described <a href="https://community.home-assistant.io/t/need-help-with-multi-zone-generic-thermostat-climate-configuration/8563">here</a>) when one heater switch is on both rooms (they will contradict). For each room I used a <a href="https://www.home-assistant.io/integrations/input_boolean/">Helper</a>, <code>input_boolean</code> switch. Trough an automation I made sure these helpers are controlling the relay switch, which controls the on-off signal to the boiler, see <a href="#automations">automations</a>.</li>
</ul>
<div class="sourceCode" id="cb2"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb2-1" data-line-number="1"><span class="fu">climate:</span></a>
<a class="sourceLine" id="cb2-2" data-line-number="2"><span class="kw">-</span> <span class="fu">platform:</span><span class="at"> generic_thermostat</span></a>
<a class="sourceLine" id="cb2-3" data-line-number="3">  <span class="fu">name:</span><span class="at"> Livingroom</span></a>
<a class="sourceLine" id="cb2-4" data-line-number="4">  <span class="fu">heater:</span><span class="at"> input_boolean.schakelaar_woonkamer</span></a>
<a class="sourceLine" id="cb2-5" data-line-number="5">  <span class="fu">target_sensor:</span><span class="at"> sensor.ds18b20_woonkamer_correctie</span></a>
<a class="sourceLine" id="cb2-6" data-line-number="6">  <span class="fu">min_temp:</span><span class="at"> 12</span></a>
<a class="sourceLine" id="cb2-7" data-line-number="7">  <span class="fu">max_temp:</span><span class="at"> 22</span></a>
<a class="sourceLine" id="cb2-8" data-line-number="8">  <span class="fu">ac_mode:</span><span class="at"> false</span></a>
<a class="sourceLine" id="cb2-9" data-line-number="9">  <span class="fu">min_cycle_duration:</span></a>
<a class="sourceLine" id="cb2-10" data-line-number="10">    <span class="fu">minutes:</span><span class="at"> 1</span></a>
<a class="sourceLine" id="cb2-11" data-line-number="11">  <span class="fu">target_temp:</span><span class="at"> 18.0</span></a>
<a class="sourceLine" id="cb2-12" data-line-number="12">  <span class="fu">cold_tolerance:</span><span class="at"> 0</span></a>
<a class="sourceLine" id="cb2-13" data-line-number="13">  <span class="fu">hot_tolerance:</span><span class="at"> 0</span></a>
<a class="sourceLine" id="cb2-14" data-line-number="14">  <span class="fu">initial_hvac_mode:</span><span class="at"> </span><span class="st">&quot;heat&quot;</span></a>
<a class="sourceLine" id="cb2-15" data-line-number="15">  <span class="fu">away_temp:</span><span class="at"> 16</span></a>
<a class="sourceLine" id="cb2-16" data-line-number="16">  <span class="fu">precision:</span><span class="at"> 0.1</span></a>
<a class="sourceLine" id="cb2-17" data-line-number="17"><span class="kw">-</span> <span class="fu">platform:</span><span class="at"> generic_thermostat</span></a>
<a class="sourceLine" id="cb2-18" data-line-number="18">  <span class="fu">name:</span><span class="at"> Bedroom</span></a>
<a class="sourceLine" id="cb2-19" data-line-number="19">  <span class="fu">heater:</span><span class="at"> input_boolean.schakelaar_slaapkamer</span></a>
<a class="sourceLine" id="cb2-20" data-line-number="20">  <span class="fu">target_sensor:</span><span class="at"> sensor.serial_sensor</span></a>
<a class="sourceLine" id="cb2-21" data-line-number="21">  <span class="fu">min_temp:</span><span class="at"> 10</span></a>
<a class="sourceLine" id="cb2-22" data-line-number="22">  <span class="fu">max_temp:</span><span class="at"> 22</span></a>
<a class="sourceLine" id="cb2-23" data-line-number="23">  <span class="fu">ac_mode:</span><span class="at"> false</span></a>
<a class="sourceLine" id="cb2-24" data-line-number="24">  <span class="fu">min_cycle_duration:</span></a>
<a class="sourceLine" id="cb2-25" data-line-number="25">    <span class="fu">minutes:</span><span class="at"> 3</span></a>
<a class="sourceLine" id="cb2-26" data-line-number="26">  <span class="fu">target_temp:</span><span class="at"> 12.0</span></a>
<a class="sourceLine" id="cb2-27" data-line-number="27">  <span class="fu">cold_tolerance:</span><span class="at"> 0</span></a>
<a class="sourceLine" id="cb2-28" data-line-number="28">  <span class="fu">hot_tolerance:</span><span class="at"> 0</span></a>
<a class="sourceLine" id="cb2-29" data-line-number="29">  <span class="fu">initial_hvac_mode:</span><span class="at"> </span><span class="st">&quot;heat&quot;</span></a>
<a class="sourceLine" id="cb2-30" data-line-number="30">  <span class="fu">away_temp:</span><span class="at"> 16</span></a>
<a class="sourceLine" id="cb2-31" data-line-number="31">  <span class="fu">precision:</span><span class="at"> 0.1</span></a></code></pre></div>
<h4 id="telegram-integration"><span class="header-section-number">0.3.3.2</span> Telegram integration</h4>
<p>I use <a href="https://telegram.org/">Telegram</a> for notifications. Currently I am using two notifications:</p>
<ul>
<li>Hours of heating during a week on Sunday at 18:00</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/110303144-53d5bc80-7ffa-11eb-98dc-0a7ae96f4686.png" /></p>
<p>“Last week there were 21 hours of heating”</p>
<ul>
<li>When the heating is automatically turned off because of a suspected open window (See Verwijzing).</li>
</ul>
<p><img src="https://user-images.githubusercontent.com/43075793/110302916-13763e80-7ffa-11eb-8579-ccd264169956.png" /></p>
<p>Translation: “Bedroom thermostat turned off bc of too slow heating up, window open?”</p>
<p>Home Assistant documentation: <a href="https://www.home-assistant.io/integrations/telegram_polling/">Telegram polling</a></p>
<div class="sourceCode" id="cb3"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb3-1" data-line-number="1"><span class="fu">telegram_bot:</span></a>
<a class="sourceLine" id="cb3-2" data-line-number="2">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> polling</span></a>
<a class="sourceLine" id="cb3-3" data-line-number="3">    <span class="fu">api_key:</span><span class="at"> secret</span></a>
<a class="sourceLine" id="cb3-4" data-line-number="4">    <span class="fu">allowed_chat_ids:</span></a>
<a class="sourceLine" id="cb3-5" data-line-number="5">      <span class="kw">-</span>  secret</a>
<a class="sourceLine" id="cb3-6" data-line-number="6"></a>
<a class="sourceLine" id="cb3-7" data-line-number="7"></a>
<a class="sourceLine" id="cb3-8" data-line-number="8"><span class="fu">notify:</span></a>
<a class="sourceLine" id="cb3-9" data-line-number="9">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> telegram</span></a>
<a class="sourceLine" id="cb3-10" data-line-number="10">    <span class="fu">name:</span><span class="at"> Telegramnotifier</span></a>
<a class="sourceLine" id="cb3-11" data-line-number="11">    <span class="fu">chat_id:</span><span class="at"> secret</span></a></code></pre></div>
<h4 id="correction-of-temperature-sensors"><span class="header-section-number">0.3.3.3</span> Correction of temperature sensors</h4>
<p>My DS18B20 sensors (<a href="https://www.home-assistant.io/integrations/onewire/">onewire</a>) need a correction to match the right temperature value. With the use of <a href="https://www.home-assistant.io/integrations/template/">template platform</a> a correction is applied to the onewire sensors. </p>
<p>See below:</p>
<div class="sourceCode" id="cb4"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb4-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb4-2" data-line-number="2"><span class="fu">sensor:</span></a>
<a class="sourceLine" id="cb4-3" data-line-number="3">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> serial</span></a>
<a class="sourceLine" id="cb4-4" data-line-number="4">    <span class="fu">serial_port:</span><span class="at"> /dev/ttyUSB0</span></a>
<a class="sourceLine" id="cb4-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> onewire</span></a>
<a class="sourceLine" id="cb4-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> time_date</span></a>
<a class="sourceLine" id="cb4-7" data-line-number="7">    <span class="fu">display_options:</span></a>
<a class="sourceLine" id="cb4-8" data-line-number="8">      <span class="kw">-</span> <span class="st">&#39;time&#39;</span></a>
<a class="sourceLine" id="cb4-9" data-line-number="9">      <span class="kw">-</span> <span class="st">&#39;date&#39;</span></a>
<a class="sourceLine" id="cb4-10" data-line-number="10">      <span class="kw">-</span> <span class="st">&#39;date_time&#39;</span></a>
<a class="sourceLine" id="cb4-11" data-line-number="11">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb4-12" data-line-number="12">    <span class="fu">sensors:</span></a>
<a class="sourceLine" id="cb4-13" data-line-number="13">      <span class="fu">ds18b20_woonkamer_correctie:</span></a>
<a class="sourceLine" id="cb4-14" data-line-number="14">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{ states(&#39;sensor.28_00000913d350_temperature&#39;)|float - 1.2}}&quot;</span></a>
<a class="sourceLine" id="cb4-15" data-line-number="15">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;Woonkamer temp&#39;</span></a>
<a class="sourceLine" id="cb4-16" data-line-number="16">        <span class="fu">unit_of_measurement:</span><span class="at"> degrees</span></a>
<a class="sourceLine" id="cb4-17" data-line-number="17">      <span class="fu">ds18b20_slaapkamer_correctie:</span></a>
<a class="sourceLine" id="cb4-18" data-line-number="18">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{ states(&#39;sensor.28_011937d1c3d1_temperature&#39;)|float - 0.6}}&quot;</span></a>
<a class="sourceLine" id="cb4-19" data-line-number="19">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;Slaapkamer temp&#39;</span></a>
<a class="sourceLine" id="cb4-20" data-line-number="20">        <span class="fu">unit_of_measurement:</span><span class="at"> degrees</span></a>
<a class="sourceLine" id="cb4-21" data-line-number="21">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> history_stats</span></a>
<a class="sourceLine" id="cb4-22" data-line-number="22">    <span class="fu">name:</span><span class="at"> Aantal minuten verwarmen laatste 7 dagen</span></a>
<a class="sourceLine" id="cb4-23" data-line-number="23">    <span class="fu">entity_id:</span><span class="at"> sensor.heating_state</span></a>
<a class="sourceLine" id="cb4-24" data-line-number="24">    <span class="fu">state:</span><span class="at"> </span><span class="st">&#39;heating&#39;</span></a>
<a class="sourceLine" id="cb4-25" data-line-number="25">    <span class="fu">type:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb4-26" data-line-number="26">    <span class="fu">end:</span><span class="at"> </span><span class="st">&#39;{{ now().replace(hour=0, minute=0, second=0) }}&#39;</span></a>
<a class="sourceLine" id="cb4-27" data-line-number="27">    <span class="fu">duration:</span></a>
<a class="sourceLine" id="cb4-28" data-line-number="28">        <span class="fu">days:</span><span class="at"> 7</span></a>
<a class="sourceLine" id="cb4-29" data-line-number="29"><span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<h4 id="trend-sensor-for-possible-open-window-detection"><span class="header-section-number">0.3.3.4</span> Trend sensor for possible open window detection</h4>
<p>Using the <a href="https://www.home-assistant.io/integrations/trend/">trend platform</a> it is checked if the temperature will rise enough while heating. If not, it can be assumed that a window is open or some other error is happening and the heating is turned off. See <a href="#automations">automations</a>. </p>
<p>The <code>min_gradient</code> value, which is the temperature rising per second, for each room is set based on trial and error. </p>
<div class="sourceCode" id="cb5"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb5-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb5-2" data-line-number="2"><span class="fu">binary_sensor:</span></a>
<a class="sourceLine" id="cb5-3" data-line-number="3">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> rpi_gpio</span></a>
<a class="sourceLine" id="cb5-4" data-line-number="4">    <span class="fu">invert_logic:</span><span class="at"> false</span></a>
<a class="sourceLine" id="cb5-5" data-line-number="5">    <span class="fu">ports:</span></a>
<a class="sourceLine" id="cb5-6" data-line-number="6">     <span class="fu">16:</span><span class="at"> Kleine Motion sensor</span></a>
<a class="sourceLine" id="cb5-7" data-line-number="7">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> rpi_gpio</span></a>
<a class="sourceLine" id="cb5-8" data-line-number="8">    <span class="fu">invert_logic:</span><span class="at"> false</span></a>
<a class="sourceLine" id="cb5-9" data-line-number="9">    <span class="fu">ports:</span></a>
<a class="sourceLine" id="cb5-10" data-line-number="10">     <span class="fu">26:</span><span class="at"> Motion sensor</span></a>
<a class="sourceLine" id="cb5-11" data-line-number="11">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> trend</span></a>
<a class="sourceLine" id="cb5-12" data-line-number="12">    <span class="fu">sensors:</span></a>
<a class="sourceLine" id="cb5-13" data-line-number="13">      <span class="fu">temp_falling_slaapkamer:</span></a>
<a class="sourceLine" id="cb5-14" data-line-number="14">        <span class="fu">entity_id:</span><span class="at"> sensor.serial_sensor</span></a>
<a class="sourceLine" id="cb5-15" data-line-number="15">        <span class="fu">sample_duration:</span><span class="at"> 150</span></a>
<a class="sourceLine" id="cb5-16" data-line-number="16">        <span class="fu">max_samples:</span><span class="at"> 3</span></a>
<a class="sourceLine" id="cb5-17" data-line-number="17">        <span class="fu">min_gradient:</span><span class="at"> 0.0003</span></a>
<a class="sourceLine" id="cb5-18" data-line-number="18">        <span class="fu">invert:</span><span class="at"> false</span></a>
<a class="sourceLine" id="cb5-19" data-line-number="19">        <span class="fu">friendly_name:</span><span class="at"> DeltaT slaapk voldoende</span></a>
<a class="sourceLine" id="cb5-20" data-line-number="20">      <span class="fu">temp_falling_woonkamer:</span></a>
<a class="sourceLine" id="cb5-21" data-line-number="21">        <span class="fu">entity_id:</span><span class="at"> sensor.ds18b20_woonkamer_correctie</span></a>
<a class="sourceLine" id="cb5-22" data-line-number="22">        <span class="fu">sample_duration:</span><span class="at"> 150</span></a>
<a class="sourceLine" id="cb5-23" data-line-number="23">        <span class="fu">max_samples:</span><span class="at"> 3</span></a>
<a class="sourceLine" id="cb5-24" data-line-number="24">        <span class="fu">min_gradient:</span><span class="at"> 0.0005</span></a>
<a class="sourceLine" id="cb5-25" data-line-number="25">        <span class="fu">invert:</span><span class="at"> false</span></a>
<a class="sourceLine" id="cb5-26" data-line-number="26">        <span class="fu">friendly_name:</span><span class="at"> DeltaT woonk voldoende</span></a>
<a class="sourceLine" id="cb5-27" data-line-number="27"> <span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<div class="sourceCode" id="cb6"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb6-1" data-line-number="1"><span class="fu">sensor:</span></a>
<a class="sourceLine" id="cb6-2" data-line-number="2">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> serial</span></a>
<a class="sourceLine" id="cb6-3" data-line-number="3">    <span class="fu">serial_port:</span><span class="at"> /dev/ttyUSB0</span></a>
<a class="sourceLine" id="cb6-4" data-line-number="4">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> onewire</span></a>
<a class="sourceLine" id="cb6-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> time_date</span></a>
<a class="sourceLine" id="cb6-6" data-line-number="6">    <span class="fu">display_options:</span></a>
<a class="sourceLine" id="cb6-7" data-line-number="7">      <span class="kw">-</span> <span class="st">&#39;time&#39;</span></a>
<a class="sourceLine" id="cb6-8" data-line-number="8">      <span class="kw">-</span> <span class="st">&#39;date&#39;</span></a>
<a class="sourceLine" id="cb6-9" data-line-number="9">      <span class="kw">-</span> <span class="st">&#39;date_time&#39;</span></a>
<a class="sourceLine" id="cb6-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb6-11" data-line-number="11">    <span class="fu">sensors:</span></a>
<a class="sourceLine" id="cb6-12" data-line-number="12">      <span class="fu">deltat_slaapkamer:</span></a>
<a class="sourceLine" id="cb6-13" data-line-number="13">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{state_attr(&#39;binary_sensor.temp_falling&#39;, &#39;gradient&#39;)|float * 1000}}&quot;</span></a>
<a class="sourceLine" id="cb6-14" data-line-number="14">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;Slaapkamer temp gradient&#39;</span></a>
<a class="sourceLine" id="cb6-15" data-line-number="15">        <span class="fu">unit_of_measurement:</span><span class="at"> </span><span class="st">&#39;graden&#39;</span></a>
<a class="sourceLine" id="cb6-16" data-line-number="16">      <span class="fu">deltat_slaapkamer_grens:</span></a>
<a class="sourceLine" id="cb6-17" data-line-number="17">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{state_attr(&#39;binary_sensor.temp_falling&#39;, &#39;min_gradient&#39;)|float * 1000}}&quot;</span><span class="er">  </span></a>
<a class="sourceLine" id="cb6-18" data-line-number="18">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;Slaapkamer min gradient&#39;</span></a>
<a class="sourceLine" id="cb6-19" data-line-number="19">        <span class="fu">unit_of_measurement:</span><span class="at"> </span><span class="st">&#39;graden&#39;</span></a>
<a class="sourceLine" id="cb6-20" data-line-number="20">      <span class="fu">deltat_woonkamer:</span></a>
<a class="sourceLine" id="cb6-21" data-line-number="21">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{state_attr(&#39;binary_sensor.temp_falling_woonkamer&#39;, &#39;gradient&#39;)|float * 1000}}&quot;</span><span class="er"> </span></a>
<a class="sourceLine" id="cb6-22" data-line-number="22">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;Woonkamer temp gradient&#39;</span></a>
<a class="sourceLine" id="cb6-23" data-line-number="23">        <span class="fu">unit_of_measurement:</span><span class="at"> </span><span class="st">&#39;graden&#39;</span></a>
<a class="sourceLine" id="cb6-24" data-line-number="24">      <span class="fu">deltat_woonkamer_grens:</span></a>
<a class="sourceLine" id="cb6-25" data-line-number="25">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{state_attr(&#39;binary_sensor.temp_falling_woonkamer&#39;, &#39;min_gradient&#39;)|float * 1000}}&quot;</span></a>
<a class="sourceLine" id="cb6-26" data-line-number="26">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;Woonkamer min gradient&#39;</span></a>
<a class="sourceLine" id="cb6-27" data-line-number="27">        <span class="fu">unit_of_measurement:</span><span class="at"> </span><span class="st">&#39;graden&#39;</span><span class="er">    </span></a>
<a class="sourceLine" id="cb6-28" data-line-number="28">      <span class="fu">heating_state:</span></a>
<a class="sourceLine" id="cb6-29" data-line-number="29">        <span class="fu">value_template:</span><span class="at"> </span><span class="st">&quot;{{state_attr(&#39;climate.woonkamer&#39;, &#39;hvac_action&#39;)}}&quot;</span></a>
<a class="sourceLine" id="cb6-30" data-line-number="30">        <span class="fu">friendly_name:</span><span class="at"> </span><span class="st">&#39;thermostaat state&#39;</span></a></code></pre></div>
<h3 id="automations"><span class="header-section-number">0.3.4</span> Automations</h3>
<h4 id="setting-temperature-time-program"><span class="header-section-number">0.3.4.1</span> Setting temperature time program</h4>
<p>Two automations per room, one for setting the desired set-temperature at bedtime and one at wake-up time. Also a helper <code>input_number.current_insteltemp_slaapkamer</code> is set with the current-set temperature. This is needed for restoring the set temperatures after restart of the system and after a manual change.  </p>
<h5 id="bedroom-set-temperature-after-bedtime"><span class="header-section-number">0.3.4.1.1</span> Bedroom set temperature after bedtime</h5>
<div class="sourceCode" id="cb7"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb7-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb7-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1587807892892&#39;</span></a>
<a class="sourceLine" id="cb7-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Slaapkamer naar instelwaarde &#39;s nachts</span></a>
<a class="sourceLine" id="cb7-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb7-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb7-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">at:</span><span class="at"> input_datetime.bedtijd</span></a>
<a class="sourceLine" id="cb7-7" data-line-number="7">    <span class="fu">platform:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb7-8" data-line-number="8">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb7-9" data-line-number="9">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb7-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">data_template:</span></a>
<a class="sourceLine" id="cb7-11" data-line-number="11">      <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb7-12" data-line-number="12">      <span class="fu">temperature:</span><span class="at"> </span><span class="st">&#39;{{ states.input_text.slaapkamer_insteltemp_nacht.state }}&#39;</span></a>
<a class="sourceLine" id="cb7-13" data-line-number="13">    <span class="fu">service:</span><span class="at"> climate.set_temperature</span></a>
<a class="sourceLine" id="cb7-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> input_number.set_value</span></a>
<a class="sourceLine" id="cb7-15" data-line-number="15">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb7-16" data-line-number="16">      <span class="fu">value:</span><span class="at"> </span><span class="st">&#39;{{ states.input_text.slaapkamer_insteltemp_nacht.state }}&#39;</span></a>
<a class="sourceLine" id="cb7-17" data-line-number="17">    <span class="fu">entity_id:</span><span class="at"> input_number.current_insteltemp_slaapkamer</span></a>
<a class="sourceLine" id="cb7-18" data-line-number="18">  <span class="fu">mode:</span><span class="at"> single</span></a>
<a class="sourceLine" id="cb7-19" data-line-number="19"><span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<h5 id="bedroom-to-set-temperature-after-bedtime"><span class="header-section-number">0.3.4.1.2</span>  Bedroom to set temperature after bedtime</h5>
<div class="sourceCode" id="cb8"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb8-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb8-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1589611935632&#39;</span></a>
<a class="sourceLine" id="cb8-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Woonkamer instelwaarde na opstaan</span></a>
<a class="sourceLine" id="cb8-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb8-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb8-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">at:</span><span class="at"> input_datetime.opstaan</span></a>
<a class="sourceLine" id="cb8-7" data-line-number="7">    <span class="fu">platform:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb8-8" data-line-number="8">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb8-9" data-line-number="9">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb8-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">data_template:</span></a>
<a class="sourceLine" id="cb8-11" data-line-number="11">      <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb8-12" data-line-number="12">      <span class="fu">temperature:</span><span class="at"> </span><span class="st">&#39;{{ states.input_text.woonk_overdag.state }}&#39;</span></a>
<a class="sourceLine" id="cb8-13" data-line-number="13">    <span class="fu">service:</span><span class="at"> climate.set_temperature</span></a>
<a class="sourceLine" id="cb8-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> input_number.set_value</span></a>
<a class="sourceLine" id="cb8-15" data-line-number="15">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb8-16" data-line-number="16">      <span class="fu">value:</span><span class="at"> </span><span class="st">&#39;{{ states.input_text.woonk_overdag.state }}&#39;</span></a>
<a class="sourceLine" id="cb8-17" data-line-number="17">    <span class="fu">entity_id:</span><span class="at"> input_number.current_insteltemp_woonkamer</span></a>
<a class="sourceLine" id="cb8-18" data-line-number="18">  <span class="fu">mode:</span><span class="at"> single</span></a>
<a class="sourceLine" id="cb8-19" data-line-number="19"><span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<h5 id="bedroom-to-set-temperature-during-day-time"><span class="header-section-number">0.3.4.1.3</span>  Bedroom to set temperature during day time</h5>
<div class="sourceCode" id="cb9"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb9-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb9-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1587807715263&#39;</span></a>
<a class="sourceLine" id="cb9-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Slaapkamer naar instelwaarde overdag</span></a>
<a class="sourceLine" id="cb9-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb9-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb9-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">at:</span><span class="at"> input_datetime.opstaan</span></a>
<a class="sourceLine" id="cb9-7" data-line-number="7">    <span class="fu">platform:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb9-8" data-line-number="8">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb9-9" data-line-number="9">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb9-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">data_template:</span></a>
<a class="sourceLine" id="cb9-11" data-line-number="11">      <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb9-12" data-line-number="12">      <span class="fu">temperature:</span><span class="at"> </span><span class="st">&#39;{{ states.input_text.slaapkamer_insteltemp_overdag.state }}&#39;</span></a>
<a class="sourceLine" id="cb9-13" data-line-number="13">    <span class="fu">service:</span><span class="at"> climate.set_temperature</span></a>
<a class="sourceLine" id="cb9-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> input_number.set_value</span></a>
<a class="sourceLine" id="cb9-15" data-line-number="15">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb9-16" data-line-number="16">      <span class="fu">value:</span><span class="at"> </span><span class="st">&#39;{{ states.input_text.slaapkamer_insteltemp_overdag.state }}&#39;</span></a>
<a class="sourceLine" id="cb9-17" data-line-number="17">    <span class="fu">entity_id:</span><span class="at"> input_number.current_insteltemp_slaapkamer</span></a>
<a class="sourceLine" id="cb9-18" data-line-number="18">  <span class="fu">mode:</span><span class="at"> single</span></a>
<a class="sourceLine" id="cb9-19" data-line-number="19"><span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<h5 id="living-room-set-temperature-after-bedtime"><span class="header-section-number">0.3.4.1.4</span>  Living room set temperature after bedtime</h5>
<pre><code>{% raw %}
- id: &#39;1587310221936&#39;
  alias: Woonkamer instelwaarde na bedtijd
  description: &#39;&#39;
  trigger:
  - at: input_datetime.bedtijd
    platform: time
  condition: []
  action:
  - data_template:
      entity_id: climate.woonkamer
      temperature: &#39;{{ states.input_text.woonk_nacht.state }}&#39;
    service: climate.set_temperature
  - service: input_number.set_value
    data:
      value: &#39;{{ states.input_text.woonk_nacht.state }}&#39;
    entity_id: input_number.current_insteltemp_woonkamer
  mode: single
{% endraw %}</code></pre>
<h3 id="presence-detection"><span class="header-section-number">0.3.5</span>  Presence detection</h3>
<p><img src="https://user-images.githubusercontent.com/43075793/117958088-f8d28580-b31a-11eb-999f-f8b17d1bf0c5.png" /></p>
<p>Presence detection is done with both the mobile phone GPS location and a PIR movement sensor in the living room. If the mobile phone was the only source for presence detection this would have been used, but since there is a PIR as well, the away modus of Generic thermostat integration isn’t used in this configuration, instead of this a helper switch input (Someone home?).  </p>
<p>I used a PIR sensor next to the smartphone, because there are some disadvantages in using smartphones only for presence detection. There are scenarios like when the battery is down or the phone is on flight mode, in which the system will think someones is home.  Also it is less suitable when having guests if the smartphone is used as only presence detection source. </p>
<p>The configuration is set so, that when the phone <code>device_tracker.pra_lx1</code> changes location from away to home or home to away the helper switch <code>input_boolean.iemandthuis</code>  is toggled. So only on <strong>state change</strong>, not on current state. Next to that, the rule is followed that if two movements aren’t being detected within the last half hour, the ‘someone home status’ <code>input_boolean.iemandthuis</code> is set to off. </p>
<p>In the evening the presence detection by the PIR motion detector should be different. It is assumed that if two times a motion is detected within half an hour during <code>evening time</code> and <code>bedtime</code>, that someone will be home during the entire night. </p>
<p>As it was noticed that the PIR used in very few occasions can have a false motion detection, the number of two motion detections was chosen. </p>
<p>This configuration set up is based on a one person household, so only one smartphone with the home assistant app running. </p>
<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Triggers</th>
<th>Between hours</th>
<th>set ‘someone home status’ to</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Two times a motion detected within 30 minutes period ‘someone home status’ is off</td>
<td>waking up and evening</td>
<td>on</td>
</tr>
<tr class="even">
<td>An half an hour with one or less motion detected while ‘someone home status’ is on</td>
<td>waking up and evening</td>
<td>off</td>
</tr>
<tr class="odd">
<td>Two times a motion detected within 30 minutes period while ‘someone home status’ is off</td>
<td>evening and bedtime</td>
<td>on</td>
</tr>
<tr class="even">
<td>Smartphone location state change to away</td>
<td>always</td>
<td>off</td>
</tr>
<tr class="odd">
<td>Smartphone location state change to home</td>
<td>always</td>
<td>on</td>
</tr>
</tbody>
</table>
<p>The following automations were set to achieve this. </p>
<h4 id="automations-for-presence-detection"><span class="header-section-number">0.3.5.1</span> Automations for presence detection</h4>
<h5 id="automation-for-minimum-of-two-movements-detected-to-trigger-other-automations"><span class="header-section-number">0.3.5.1.1</span> Automation for minimum of two movements detected to trigger other automations</h5>
<p>Set two input_datetime fields on a motion detection with the PIR. One input field is set to date/time of the last movement detected and on the next motion this value is passed to one-before-last-input boolean.</p>
<p>This is used, bc it is desired that only when a minimum of two movements are detected in the last 30 minutes the status of some one should be turned to ‘on’.</p>
<div class="sourceCode" id="cb11"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb11-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1606672315270&#39;</span></a>
<a class="sourceLine" id="cb11-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Bewegingssensor last </span></a>
<a class="sourceLine" id="cb11-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb11-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb11-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb11-6" data-line-number="6">    <span class="fu">entity_id:</span><span class="at"> binary_sensor.motion_sensor</span></a>
<a class="sourceLine" id="cb11-7" data-line-number="7">    <span class="fu">to:</span><span class="at"> </span><span class="st">&#39;on&#39;</span></a>
<a class="sourceLine" id="cb11-8" data-line-number="8">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb11-9" data-line-number="9">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb11-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> input_datetime.set_datetime</span></a>
<a class="sourceLine" id="cb11-11" data-line-number="11">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb11-12" data-line-number="12">      <span class="fu">datetime:</span><span class="at"> </span><span class="st">&#39;{{states(&#39;</span><span class="er">&#39;input_datetime.beweginglaatst_0&#39;&#39;)}}&#39;</span></a>
<a class="sourceLine" id="cb11-13" data-line-number="13">    <span class="fu">entity_id:</span><span class="at"> input_datetime.bewegingeennalaatst_1</span></a>
<a class="sourceLine" id="cb11-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> input_datetime.set_datetime</span></a>
<a class="sourceLine" id="cb11-15" data-line-number="15">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb11-16" data-line-number="16">      <span class="fu">datetime:</span><span class="at"> </span><span class="st">&#39;{{ now().strftime(&#39;</span><span class="er">&#39;%Y-%m-%d %H:%M:%S&#39;&#39;) }}&#39;</span></a>
<a class="sourceLine" id="cb11-17" data-line-number="17">    <span class="fu">entity_id:</span><span class="at"> input_datetime.beweginglaatst_0</span></a>
<a class="sourceLine" id="cb11-18" data-line-number="18">  <span class="fu">mode:</span><span class="at"> single</span></a></code></pre></div>
<h5 id="reset-the-one-before-last-input-boolean-31-minutes-before-waking-time"><span class="header-section-number">0.3.5.1.2</span> Reset the one-before-last-input boolean 31 minutes before waking time</h5>
<p>Needed for the someone home status to turn on immediately when entering the living room in the morning, otherwise first two motions need to be detected, which can take a while. </p>
<div class="sourceCode" id="cb12"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb12-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1606905142912&#39;</span></a>
<a class="sourceLine" id="cb12-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Reset 1 na laatste beweging 31 min voor opstaan</span></a>
<a class="sourceLine" id="cb12-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb12-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb12-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> time_pattern</span></a>
<a class="sourceLine" id="cb12-6" data-line-number="6">    <span class="fu">seconds:</span><span class="at"> </span><span class="st">&#39;30&#39;</span></a>
<a class="sourceLine" id="cb12-7" data-line-number="7">  <span class="fu">condition:</span></a>
<a class="sourceLine" id="cb12-8" data-line-number="8">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb12-9" data-line-number="9">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{/% set current_time = now().hour * 60 + now().minute %}</span></a>
<a class="sourceLine" id="cb12-10" data-line-number="10"></a>
<a class="sourceLine" id="cb12-11" data-line-number="11"><span class="st">      {/% set opstaan_hour, opstaan_minute, opstaan_second = states(&#39;</span><span class="er">&#39;input_datetime.opstaan&#39;&#39;).split(&#39;&#39;:&#39;&#39;)</span></a>
<a class="sourceLine" id="cb12-12" data-line-number="12">      %}</a>
<a class="sourceLine" id="cb12-13" data-line-number="13"></a>
<a class="sourceLine" id="cb12-14" data-line-number="14">      <span class="kw">{</span>/% set opstaan_time = opstaan_hour | int * 60 + opstaan_minute | int %<span class="kw">}</span></a>
<a class="sourceLine" id="cb12-15" data-line-number="15"></a>
<a class="sourceLine" id="cb12-16" data-line-number="16">      <span class="kw">{</span>{ current_time == opstaan_time - 32 <span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb12-17" data-line-number="17"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb12-18" data-line-number="18"><span class="st">  - service: input_datetime.set_datetime</span></a>
<a class="sourceLine" id="cb12-19" data-line-number="19"><span class="st">    data:</span></a>
<a class="sourceLine" id="cb12-20" data-line-number="20"><span class="st">      datetime: &#39;</span><span class="kw">{</span><span class="fu">{now().strftime(&#39;&#39;%Y-%m-%d %H:</span><span class="at">%M:%S&#39;&#39;)</span><span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb12-21" data-line-number="21"><span class="st">    entity_id: input_datetime.bewegingeennalaatst_1</span></a>
<a class="sourceLine" id="cb12-22" data-line-number="22"><span class="st">  mode: restart</span></a></code></pre></div>
<h5 id="automation-to-turn-on-someone-home-status"><span class="header-section-number">0.3.5.1.3</span> Automation to turn on someone home status</h5>
<div class="sourceCode" id="cb13"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb13-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb13-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1587319960331&#39;</span></a>
<a class="sourceLine" id="cb13-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Turn on someone home status</span></a>
<a class="sourceLine" id="cb13-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb13-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb13-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">entity_id:</span><span class="at"> device_tracker.pra_lx1</span></a>
<a class="sourceLine" id="cb13-7" data-line-number="7">    <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb13-8" data-line-number="8">    <span class="fu">to:</span><span class="at"> home</span></a>
<a class="sourceLine" id="cb13-9" data-line-number="9">  <span class="kw">-</span> <span class="fu">entity_id:</span><span class="at"> binary_sensor.motion_sensor</span></a>
<a class="sourceLine" id="cb13-10" data-line-number="10">    <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb13-11" data-line-number="11">    <span class="fu">to:</span><span class="at"> </span><span class="st">&#39;on&#39;</span></a>
<a class="sourceLine" id="cb13-12" data-line-number="12">  <span class="fu">condition:</span></a>
<a class="sourceLine" id="cb13-13" data-line-number="13">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb13-14" data-line-number="14">    <span class="fu">entity_id:</span><span class="at"> input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb13-15" data-line-number="15">    <span class="fu">state:</span><span class="at"> </span><span class="st">&#39;off&#39;</span></a>
<a class="sourceLine" id="cb13-16" data-line-number="16">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb13-17" data-line-number="17">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb13-18" data-line-number="18">    <span class="fu">entity_id:</span><span class="at"> input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb13-19" data-line-number="19">    <span class="fu">service:</span><span class="at"> input_boolean.turn_on</span></a>
<a class="sourceLine" id="cb13-20" data-line-number="20">  <span class="fu">mode:</span><span class="at"> single</span></a>
<a class="sourceLine" id="cb13-21" data-line-number="21"><span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<h5 id="automation-during-evening-and-getting-up"><span class="header-section-number">0.3.5.1.4</span>  Automation during evening and getting up </h5>
<pre><code>{% raw %}
- id: &#39;1587404974211&#39;
  alias: Aanwezigheid detectie avond tot opstaan
  description: &#39;&#39;
  trigger:
  - platform: template
    value_template: &#39;{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      == 300 }}&#39;
  condition:
  - condition: state
    entity_id: input_boolean.iemandthuis
    state: &#39;off&#39;
  - before: input_datetime.opstaan
    condition: time
    after: input_datetime.avond
  - condition: template
    value_template: &#39;{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()
      &lt; 300 }}&#39;
  action:
  - data: {}
    entity_id: input_boolean.iemandthuis
    service: input_boolean.turn_on
  mode: single
- id: &#39;1587373774458&#39;
  alias: Thermostaat aan bij iemand thuis
  description: &#39;&#39;
  trigger:
  - entity_id: input_boolean.iemandthuis
    platform: state
    to: &#39;on&#39;
  condition: []
  action:
  - data: {}
    entity_id: climate.woonkamer
    service: climate.turn_on
  - data: {}
    entity_id: climate.slaapkamer
    service: climate.turn_on
{% endraw %}</code></pre>
<h5 id="automation-between-getting-up-time-and-evening-time"><span class="header-section-number">0.3.5.1.5</span>  Automation between getting up time and evening time</h5>
<div class="sourceCode" id="cb15"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb15-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb15-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1587319961411&#39;</span></a>
<a class="sourceLine" id="cb15-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Behaviour of motion sensor living room between wakeup time and evening time</span></a>
<a class="sourceLine" id="cb15-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb15-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb15-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb15-7" data-line-number="7">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{{ (states.sensor.time.last_changed - states.input_datetime.bewegingeennalaatst_1.last_changed).total_seconds()</span></a>
<a class="sourceLine" id="cb15-8" data-line-number="8"><span class="st">      &gt; 1800 }}</span></a>
<a class="sourceLine" id="cb15-9" data-line-number="9"></a>
<a class="sourceLine" id="cb15-10" data-line-number="10"><span class="st">      &#39;</span></a>
<a class="sourceLine" id="cb15-11" data-line-number="11">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb15-12" data-line-number="12">    <span class="fu">entity_id:</span><span class="at"> person.johan</span></a>
<a class="sourceLine" id="cb15-13" data-line-number="13">    <span class="fu">to:</span><span class="at"> not_home</span></a>
<a class="sourceLine" id="cb15-14" data-line-number="14">  <span class="fu">condition:</span></a>
<a class="sourceLine" id="cb15-15" data-line-number="15">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb15-16" data-line-number="16">    <span class="fu">entity_id:</span><span class="at"> input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb15-17" data-line-number="17">    <span class="fu">state:</span><span class="at"> </span><span class="st">&#39;on&#39;</span></a>
<a class="sourceLine" id="cb15-18" data-line-number="18">  <span class="kw">-</span> <span class="fu">before:</span><span class="at"> input_datetime.bedtijd</span></a>
<a class="sourceLine" id="cb15-19" data-line-number="19">    <span class="fu">condition:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb15-20" data-line-number="20">    <span class="fu">after:</span><span class="at"> input_datetime.opstaan</span></a>
<a class="sourceLine" id="cb15-21" data-line-number="21">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb15-22" data-line-number="22">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb15-23" data-line-number="23">    <span class="fu">entity_id:</span><span class="at"> input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb15-24" data-line-number="24">    <span class="fu">service:</span><span class="at"> input_boolean.turn_off</span></a>
<a class="sourceLine" id="cb15-25" data-line-number="25">  <span class="fu">mode:</span><span class="at"> single</span></a>
<a class="sourceLine" id="cb15-26" data-line-number="26"><span class="kw">{</span>% endraw %<span class="kw">}</span></a></code></pre></div>
<h5 id="behavior-based-on-smart-phone-location-with-home-assistant-app"><span class="header-section-number">0.3.5.1.6</span> Behavior based on smart phone location with Home Assistant app</h5>
<pre><code>
- id: &#39;1606221709908&#39;
  alias: Smartphone thuiskomen / weggaan
  description: &#39;&#39;
  trigger:
  - platform: state
    entity_id: person.johan
    to: not_home
  condition: []
  action:
  - service: input_boolean.turn_off
    data: {}
    entity_id: input_boolean.iemandthuis
  - wait_for_trigger:
    - platform: state
      entity_id: person.johan
      to: home
  - service: input_boolean.turn_on
    data: {}
    entity_id: input_boolean.iemandthuis
  mode: restart</code></pre>
<h5 id="turning-off-thermostat-when-someone-home-status-is-off"><span class="header-section-number">0.3.5.1.7</span>  Turning off thermostat when Someone home status is off</h5>
<div class="sourceCode" id="cb17"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb17-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1587373899805&#39;</span></a>
<a class="sourceLine" id="cb17-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Thermostaat uit bij niemand thuis</span></a>
<a class="sourceLine" id="cb17-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb17-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb17-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">entity_id:</span><span class="at"> input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb17-6" data-line-number="6">    <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb17-7" data-line-number="7">    <span class="fu">to:</span><span class="at"> </span><span class="st">&#39;off&#39;</span></a>
<a class="sourceLine" id="cb17-8" data-line-number="8">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb17-9" data-line-number="9">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb17-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb17-11" data-line-number="11">    <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb17-12" data-line-number="12">    <span class="fu">service:</span><span class="at"> climate.turn_off</span></a>
<a class="sourceLine" id="cb17-13" data-line-number="13">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb17-14" data-line-number="14">    <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb17-15" data-line-number="15">    <span class="fu">service:</span><span class="at"> climate.turn_off </span></a></code></pre></div>
<h3 id="heat-for-5-minutes-straight"><span class="header-section-number">0.3.6</span> Heat for 5 minutes straight</h3>
<h5 id="automation"><span class="header-section-number">0.3.6.0.1</span> Automation:</h5>
<div class="sourceCode" id="cb18"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb18-1" data-line-number="1"></a>
<a class="sourceLine" id="cb18-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1588717917351&#39;</span></a>
<a class="sourceLine" id="cb18-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> 5 minuten verwarmen als schakelaar aan</span></a>
<a class="sourceLine" id="cb18-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb18-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb18-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">entity_id:</span><span class="at"> input_boolean.30_min_verwarmen_schakelaar</span></a>
<a class="sourceLine" id="cb18-7" data-line-number="7">    <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb18-8" data-line-number="8">    <span class="fu">to:</span><span class="at"> </span><span class="st">&#39;on&#39;</span></a>
<a class="sourceLine" id="cb18-9" data-line-number="9">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb18-10" data-line-number="10">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb18-11" data-line-number="11">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb18-12" data-line-number="12">    <span class="fu">entity_id:</span><span class="at"> switch.relay</span></a>
<a class="sourceLine" id="cb18-13" data-line-number="13">    <span class="fu">service:</span><span class="at"> switch.turn_on</span></a>
<a class="sourceLine" id="cb18-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">timeout:</span><span class="at"> 00:05</span></a>
<a class="sourceLine" id="cb18-15" data-line-number="15">    <span class="fu">wait_template:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb18-16" data-line-number="16">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb18-17" data-line-number="17">    <span class="fu">entity_id:</span><span class="at"> switch.relay</span></a>
<a class="sourceLine" id="cb18-18" data-line-number="18">    <span class="fu">service:</span><span class="at"> switch.turn_off</span></a>
<a class="sourceLine" id="cb18-19" data-line-number="19">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> input_boolean.turn_off</span></a>
<a class="sourceLine" id="cb18-20" data-line-number="20">    <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb18-21" data-line-number="21">    <span class="fu">entity_id:</span><span class="at"> input_boolean.30_min_verwarmen_schakelaar</span></a>
<a class="sourceLine" id="cb18-22" data-line-number="22">  <span class="fu">mode:</span><span class="at"> single</span></a></code></pre></div>
<h3 id="window-open-detection-based-on-speed-of-temperature-rise"><span class="header-section-number">0.3.7</span> Window open detection (based on speed of temperature rise)</h3>
<p>Uses the <a href="https://www.home-assistant.io/integrations/trend/">Trend sensor</a> to make the <code>binary_sensor.temp_falling</code></p>
<p>After 300 seconds of heating without reaching the treshold of de trend sensor, the relay switch is turned off and sends a Telegram notification. </p>
<div class="sourceCode" id="cb19"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb19-1" data-line-number="1"></a>
<a class="sourceLine" id="cb19-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1588799650928&#39;</span></a>
<a class="sourceLine" id="cb19-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Relay beveiging raam open</span></a>
<a class="sourceLine" id="cb19-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb19-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb19-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> time_pattern</span></a>
<a class="sourceLine" id="cb19-7" data-line-number="7">    <span class="fu">minutes:</span><span class="at"> </span><span class="st">&#39;5&#39;</span></a>
<a class="sourceLine" id="cb19-8" data-line-number="8">  <span class="fu">condition:</span></a>
<a class="sourceLine" id="cb19-9" data-line-number="9">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb19-10" data-line-number="10">    <span class="fu">entity_id:</span><span class="at"> binary_sensor.temp_falling</span></a>
<a class="sourceLine" id="cb19-11" data-line-number="11">    <span class="fu">state:</span><span class="at"> </span><span class="st">&#39;off&#39;</span></a>
<a class="sourceLine" id="cb19-12" data-line-number="12">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb19-13" data-line-number="13">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{{state_attr(&#39;</span><span class="er">&#39;climate.slaapkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) | float</span></a>
<a class="sourceLine" id="cb19-14" data-line-number="14">      &gt; state_attr(<span class="st">&#39;&#39;</span>climate.slaapkamer<span class="st">&#39;&#39;</span>, <span class="st">&#39;&#39;</span>current_temperature<span class="st">&#39;&#39;</span>)}}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb19-15" data-line-number="15"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb19-16" data-line-number="16"><span class="st">    entity_id: switch.relay</span></a>
<a class="sourceLine" id="cb19-17" data-line-number="17"><span class="st">    state: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb19-18" data-line-number="18"><span class="st">  - condition: template</span></a>
<a class="sourceLine" id="cb19-19" data-line-number="19"><span class="st">    value_template: &#39;</span><span class="kw">{</span>{ (states.sensor.time.last_changed  - states.switch.relay.last_changed).total_seconds()</a>
<a class="sourceLine" id="cb19-20" data-line-number="20">      &gt; 310 <span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb19-21" data-line-number="21"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb19-22" data-line-number="22"><span class="st">  - service: telegram_bot.send_message</span></a>
<a class="sourceLine" id="cb19-23" data-line-number="23"><span class="st">    data:</span></a>
<a class="sourceLine" id="cb19-24" data-line-number="24"><span class="st">      message: Slaapkamer thermostaat uitgeschakeld ivm te langzame opwarming, raam</span></a>
<a class="sourceLine" id="cb19-25" data-line-number="25"><span class="st">        open?</span></a>
<a class="sourceLine" id="cb19-26" data-line-number="26"><span class="st">      target: [chat-number]</span></a>
<a class="sourceLine" id="cb19-27" data-line-number="27"><span class="st">  - data: {}</span></a>
<a class="sourceLine" id="cb19-28" data-line-number="28"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb19-29" data-line-number="29"><span class="st">    service: climate.turn_off</span></a>
<a class="sourceLine" id="cb19-30" data-line-number="30"><span class="st">  - delay: 00:45:00</span></a>
<a class="sourceLine" id="cb19-31" data-line-number="31"><span class="st">  - service: climate.turn_on</span></a>
<a class="sourceLine" id="cb19-32" data-line-number="32"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb19-33" data-line-number="33"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb19-34" data-line-number="34"><span class="st">  mode: single</span></a></code></pre></div>
<div class="sourceCode" id="cb20"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb20-1" data-line-number="1"></a>
<a class="sourceLine" id="cb20-2" data-line-number="2"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1601215001454&#39;</span></a>
<a class="sourceLine" id="cb20-3" data-line-number="3">  <span class="fu">alias:</span><span class="at"> Relay beveiging raam open (woonkamer)</span></a>
<a class="sourceLine" id="cb20-4" data-line-number="4">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb20-5" data-line-number="5">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb20-6" data-line-number="6">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb20-7" data-line-number="7">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{{ (states.sensor.time.last_changed  - states.switch.relay.last_changed).total_seconds()</span></a>
<a class="sourceLine" id="cb20-8" data-line-number="8"><span class="st">      &gt; 400 }}&#39;</span></a>
<a class="sourceLine" id="cb20-9" data-line-number="9">  <span class="fu">condition:</span></a>
<a class="sourceLine" id="cb20-10" data-line-number="10">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb20-11" data-line-number="11">    <span class="fu">entity_id:</span><span class="at"> automation.slaapkamer_overgang_off_on_set_schakelaar</span></a>
<a class="sourceLine" id="cb20-12" data-line-number="12">    <span class="fu">state:</span><span class="at"> </span><span class="st">&#39;on&#39;</span></a>
<a class="sourceLine" id="cb20-13" data-line-number="13">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb20-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb20-15" data-line-number="15">    <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb20-16" data-line-number="16">    <span class="fu">service:</span><span class="at"> climate.turn_off</span></a>
<a class="sourceLine" id="cb20-17" data-line-number="17">  <span class="kw">-</span> <span class="fu">delay:</span><span class="at"> 00:59:00</span></a>
<a class="sourceLine" id="cb20-18" data-line-number="18">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> climate.turn_on</span></a>
<a class="sourceLine" id="cb20-19" data-line-number="19">    <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb20-20" data-line-number="20">    <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb20-21" data-line-number="21">  <span class="fu">mode:</span><span class="at"> single</span></a></code></pre></div>
<h5 id="revert-back-to-programmed-set-temperature-after-manual-change"><span class="header-section-number">0.3.7.0.1</span> Revert back to programmed set temperature after manual change  </h5>
<p>According to <code>input_datetime.duur_manuele_verhoging</code> value a timer is started after which the set temperature will revert back to set temperature according to program. </p>
<p>Uses the <a href="https://www.home-assistant.io/integrations/timer/">Timer integration</a></p>
<div class="sourceCode" id="cb21"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb21-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1604920070266&#39;</span></a>
<a class="sourceLine" id="cb21-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Countdown bij manual wijziging</span></a>
<a class="sourceLine" id="cb21-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb21-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb21-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb21-6" data-line-number="6">    <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb21-7" data-line-number="7">    <span class="fu">attribute:</span><span class="at"> temperature</span></a>
<a class="sourceLine" id="cb21-8" data-line-number="8">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> state</span></a>
<a class="sourceLine" id="cb21-9" data-line-number="9">    <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb21-10" data-line-number="10">    <span class="fu">attribute:</span><span class="at"> temperature</span></a>
<a class="sourceLine" id="cb21-11" data-line-number="11">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb21-12" data-line-number="12">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb21-13" data-line-number="13">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> timer.start</span></a>
<a class="sourceLine" id="cb21-14" data-line-number="14">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb21-15" data-line-number="15">      <span class="fu">duration:</span><span class="at"> </span><span class="st">&#39;{{ states.input_datetime.duur_manuele_verhoging.state }}&#39;</span></a>
<a class="sourceLine" id="cb21-16" data-line-number="16">    <span class="fu">entity_id:</span><span class="at"> timer.countdown</span></a>
<a class="sourceLine" id="cb21-17" data-line-number="17">  <span class="kw">-</span> <span class="fu">delay:</span><span class="at"> </span><span class="st">&#39;{{ states.input_datetime.duur_manuele_verhoging.state }}&#39;</span></a>
<a class="sourceLine" id="cb21-18" data-line-number="18">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> climate.set_temperature</span></a>
<a class="sourceLine" id="cb21-19" data-line-number="19">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb21-20" data-line-number="20">      <span class="fu">temperature:</span><span class="at"> </span><span class="st">&#39;{{ states.input_number.current_insteltemp_slaapkamer.state  }}&#39;</span></a>
<a class="sourceLine" id="cb21-21" data-line-number="21">    <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb21-22" data-line-number="22">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> climate.set_temperature</span></a>
<a class="sourceLine" id="cb21-23" data-line-number="23">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb21-24" data-line-number="24">      <span class="fu">temperature:</span><span class="at"> </span><span class="st">&#39;{{ states.input_number.current_insteltemp_woonkamer.state  }}&#39;</span></a>
<a class="sourceLine" id="cb21-25" data-line-number="25">    <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb21-26" data-line-number="26">  <span class="fu">mode:</span><span class="at"> restart</span></a></code></pre></div>
<h5 id="telegram-notification-hours-of-heating-past-week-on-sunday"><span class="header-section-number">0.3.7.0.2</span> Telegram notification hours of heating past week on Sunday</h5>
<div class="sourceCode" id="cb22"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb22-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1604938488226&#39;</span></a>
<a class="sourceLine" id="cb22-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Notificatie aantal uren verwarmen in week</span></a>
<a class="sourceLine" id="cb22-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb22-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb22-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb22-6" data-line-number="6">    <span class="fu">at:</span><span class="at"> </span><span class="st">&#39;18:00&#39;</span></a>
<a class="sourceLine" id="cb22-7" data-line-number="7">  <span class="fu">condition:</span></a>
<a class="sourceLine" id="cb22-8" data-line-number="8">  <span class="kw">-</span> <span class="fu">condition:</span><span class="at"> time</span></a>
<a class="sourceLine" id="cb22-9" data-line-number="9">    <span class="fu">weekday:</span></a>
<a class="sourceLine" id="cb22-10" data-line-number="10">    <span class="kw">-</span> sun</a>
<a class="sourceLine" id="cb22-11" data-line-number="11">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb22-12" data-line-number="12">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> telegram_bot.send_message</span></a>
<a class="sourceLine" id="cb22-13" data-line-number="13">    <span class="fu">data:</span></a>
<a class="sourceLine" id="cb22-14" data-line-number="14">      <span class="fu">message:</span><span class="at"> De afgelopen week is er {{ states.sensor.aantal_minuten_verwarmen_laatste_7_dagen.state</span></a>
<a class="sourceLine" id="cb22-15" data-line-number="15">        | int}} uren verwarmd.</a>
<a class="sourceLine" id="cb22-16" data-line-number="16">  <span class="fu">mode:</span><span class="at"> single</span></a></code></pre></div>
<h5 id="turn-of-heating-when-there-is-no-signal-of-ds18b20-temperature-sensor"><span class="header-section-number">0.3.7.0.3</span> Turn of heating when there is no signal of DS18B20 temperature sensor</h5>
<p>It occasionally happens that there is no signal of the DS18B20 temperature sensor or that by mistake the USB cable gets unplugged. The displayed temperature then can get below set temperature and will trigger heating while not really desired. To avoid this an automation is set to turn off. </p>
<div class="sourceCode" id="cb23"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb23-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1606221459609&#39;</span></a>
<a class="sourceLine" id="cb23-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Beveiliging uitvallen temp sensor</span></a>
<a class="sourceLine" id="cb23-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb23-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb23-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> numeric_state</span></a>
<a class="sourceLine" id="cb23-6" data-line-number="6">    <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb23-7" data-line-number="7">    <span class="fu">attribute:</span><span class="at"> current_temperature</span></a>
<a class="sourceLine" id="cb23-8" data-line-number="8">    <span class="fu">below:</span><span class="at"> </span><span class="st">&#39;10&#39;</span></a>
<a class="sourceLine" id="cb23-9" data-line-number="9">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> numeric_state</span></a>
<a class="sourceLine" id="cb23-10" data-line-number="10">    <span class="fu">entity_id:</span><span class="at"> sensor.ds18b20_woonkamer_correctie</span></a>
<a class="sourceLine" id="cb23-11" data-line-number="11">    <span class="fu">below:</span><span class="at"> </span><span class="st">&#39;10&#39;</span></a>
<a class="sourceLine" id="cb23-12" data-line-number="12">  <span class="fu">condition:</span><span class="at"> </span><span class="kw">[]</span></a>
<a class="sourceLine" id="cb23-13" data-line-number="13">  <span class="fu">action:</span></a>
<a class="sourceLine" id="cb23-14" data-line-number="14">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> climate.turn_off</span></a>
<a class="sourceLine" id="cb23-15" data-line-number="15">    <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb23-16" data-line-number="16">    <span class="fu">entity_id:</span><span class="at"> climate.slaapkamer</span></a>
<a class="sourceLine" id="cb23-17" data-line-number="17">  <span class="kw">-</span> <span class="fu">service:</span><span class="at"> climate.turn_off</span></a>
<a class="sourceLine" id="cb23-18" data-line-number="18">    <span class="fu">data:</span><span class="at"> </span><span class="kw">{}</span></a>
<a class="sourceLine" id="cb23-19" data-line-number="19">    <span class="fu">entity_id:</span><span class="at"> climate.woonkamer</span></a>
<a class="sourceLine" id="cb23-20" data-line-number="20">  <span class="fu">mode:</span><span class="at"> single</span></a></code></pre></div>
<h5 id="controlling-two-generic-thermostat-entities"><span class="header-section-number">0.3.7.0.4</span> Controlling two generic thermostat entities</h5>
<p>The <a href="https://www.home-assistant.io/integrations/generic_thermostat/">generic thermostat integration</a> is equipped to work only with one temperature sensor. You can run two instances of the generic thermostat integration. However when the heater option is set to the same switch, then when one thermostat is turned on, the other will automatically turn on too (bc they use the same switch,  generic thermostat is programmed like that). This sometimes can lead to situations in which the thermostat of a room will turn on while cold air is flowing in because of a open window. </p>
<p>Therefore a couple of input_booleans are created and are set as heater switch. Via an automation the relay switch will be turned on if one of these input_booleans are turned on. </p>
<p>Also the someone status <code>input_boolean.iemandthuis</code> is taken into account </p>
<h6 id="living-room-thermostat-turn-on"><span class="header-section-number">0.3.7.0.4.1</span> Living room thermostat turn on</h6>
<div class="sourceCode" id="cb24"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb24-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1606337912735&#39;</span></a>
<a class="sourceLine" id="cb24-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> Woonkamer thermostaat aan</span></a>
<a class="sourceLine" id="cb24-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb24-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb24-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb24-6" data-line-number="6">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{{state_attr(&#39;</span><span class="er">&#39;climate.woonkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) &gt; state_attr(&#39;&#39;climate.woonkamer&#39;&#39;,</span></a>
<a class="sourceLine" id="cb24-7" data-line-number="7">      <span class="st">&#39;&#39;</span>current_temperature<span class="st">&#39;&#39;</span>)}}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb24-8" data-line-number="8"><span class="st">  - platform: template</span></a>
<a class="sourceLine" id="cb24-9" data-line-number="9"><span class="st">    value_template: &#39;</span><span class="kw">{</span>{ (states.sensor.time.last_changed - states.input_boolean.iemandthuis.last_changed).total_seconds()</a>
<a class="sourceLine" id="cb24-10" data-line-number="10">      &gt; 5 <span class="kw">}</span>}</a>
<a class="sourceLine" id="cb24-11" data-line-number="11"></a>
<a class="sourceLine" id="cb24-12" data-line-number="12">      <span class="st">&#39;</span></a>
<a class="sourceLine" id="cb24-13" data-line-number="13"><span class="st">  - platform: time_pattern</span></a>
<a class="sourceLine" id="cb24-14" data-line-number="14"><span class="st">    minutes: &#39;</span>3<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb24-15" data-line-number="15"><span class="st">  condition:</span></a>
<a class="sourceLine" id="cb24-16" data-line-number="16"><span class="st">  - condition: template</span></a>
<a class="sourceLine" id="cb24-17" data-line-number="17"><span class="st">    value_template: &#39;</span><span class="kw">{</span>{state_attr(&#39;&#39;climate.woonkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;)  &gt; state_attr(&#39;&#39;climate.woonkamer&#39;&#39;,</a>
<a class="sourceLine" id="cb24-18" data-line-number="18">      &#39;&#39;current_temperature&#39;&#39;)<span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb24-19" data-line-number="19"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb24-20" data-line-number="20"><span class="st">    entity_id: input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb24-21" data-line-number="21"><span class="st">    state: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb24-22" data-line-number="22"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb24-23" data-line-number="23"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb24-24" data-line-number="24"><span class="st">    state: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb24-25" data-line-number="25"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb24-26" data-line-number="26"><span class="st">  - service: climate.turn_on</span></a>
<a class="sourceLine" id="cb24-27" data-line-number="27"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb24-28" data-line-number="28"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb24-29" data-line-number="29"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb24-30" data-line-number="30"><span class="st">  max: 10</span></a></code></pre></div>
<h6 id="bedroom-thermostat-turn-on-off"><span class="header-section-number">0.3.7.0.4.2</span> Bedroom thermostat turn on / off </h6>
<div class="sourceCode" id="cb25"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb25-1" data-line-number="1"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1606338268883&#39;</span></a>
<a class="sourceLine" id="cb25-2" data-line-number="2">  <span class="fu">alias:</span><span class="at"> </span><span class="st">&#39;Slaapkamer thermostaat aan &#39;</span></a>
<a class="sourceLine" id="cb25-3" data-line-number="3">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb25-4" data-line-number="4">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb25-5" data-line-number="5">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb25-6" data-line-number="6">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{{state_attr(&#39;</span><span class="er">&#39;climate.slaapkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) &gt; state_attr(&#39;&#39;climate.slaapkamer&#39;&#39;,</span></a>
<a class="sourceLine" id="cb25-7" data-line-number="7">      <span class="st">&#39;&#39;</span>current_temperature<span class="st">&#39;&#39;</span>)}}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb25-8" data-line-number="8"><span class="st">  - platform: time_pattern</span></a>
<a class="sourceLine" id="cb25-9" data-line-number="9"><span class="st">    minutes: &#39;</span>3<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb25-10" data-line-number="10"><span class="st">  - platform: template</span></a>
<a class="sourceLine" id="cb25-11" data-line-number="11"><span class="st">    value_template: &#39;</span></a>
<a class="sourceLine" id="cb25-12" data-line-number="12"></a>
<a class="sourceLine" id="cb25-13" data-line-number="13">      <span class="kw">{</span>{ (states.sensor.time.last_changed - states.input_boolean.iemandthuis.last_changed).total_seconds()</a>
<a class="sourceLine" id="cb25-14" data-line-number="14">      &gt; 5 <span class="kw">}</span>}</a>
<a class="sourceLine" id="cb25-15" data-line-number="15"></a>
<a class="sourceLine" id="cb25-16" data-line-number="16">      <span class="st">&#39;</span></a>
<a class="sourceLine" id="cb25-17" data-line-number="17"><span class="st">  condition:</span></a>
<a class="sourceLine" id="cb25-18" data-line-number="18"><span class="st">  - condition: template</span></a>
<a class="sourceLine" id="cb25-19" data-line-number="19"><span class="st">    value_template: &#39;</span><span class="kw">{</span>{state_attr(&#39;&#39;climate.slaapkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) &gt; state_attr(&#39;&#39;climate.slaapkamer&#39;&#39;,</a>
<a class="sourceLine" id="cb25-20" data-line-number="20">      &#39;&#39;current_temperature&#39;&#39;)<span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb25-21" data-line-number="21"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb25-22" data-line-number="22"><span class="st">    entity_id: input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb25-23" data-line-number="23"><span class="st">    state: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb25-24" data-line-number="24"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb25-25" data-line-number="25"><span class="st">  - service: climate.turn_on</span></a>
<a class="sourceLine" id="cb25-26" data-line-number="26"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb25-27" data-line-number="27"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb25-28" data-line-number="28"><span class="st">  - wait_for_trigger:</span></a>
<a class="sourceLine" id="cb25-29" data-line-number="29"><span class="st">    - platform: template</span></a>
<a class="sourceLine" id="cb25-30" data-line-number="30"><span class="st">      value_template: &#39;</span><span class="kw">{</span>{state_attr(&#39;&#39;climate.slaapkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) &lt; state_attr(&#39;&#39;climate.slaapkamer&#39;&#39;,</a>
<a class="sourceLine" id="cb25-31" data-line-number="31">        &#39;&#39;current_temperature&#39;&#39;)<span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb25-32" data-line-number="32"><span class="st">  - service: climate.turn_off</span></a>
<a class="sourceLine" id="cb25-33" data-line-number="33"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb25-34" data-line-number="34"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb25-35" data-line-number="35"><span class="st">  mode: restart</span></a></code></pre></div>
<h5 id="living-room-turn-thermostat-off"><span class="header-section-number">0.3.7.0.5</span>  Living room turn thermostat off</h5>
<div class="sourceCode" id="cb26"><pre class="sourceCode yaml"><code class="sourceCode yaml"><a class="sourceLine" id="cb26-1" data-line-number="1"><span class="kw">{</span>% raw %<span class="kw">}</span></a>
<a class="sourceLine" id="cb26-2" data-line-number="2"></a>
<a class="sourceLine" id="cb26-3" data-line-number="3"><span class="kw">-</span> <span class="fu">id:</span><span class="at"> </span><span class="st">&#39;1606839006446&#39;</span></a>
<a class="sourceLine" id="cb26-4" data-line-number="4">  <span class="fu">alias:</span><span class="at"> Woonkamer thermostaat uit</span></a>
<a class="sourceLine" id="cb26-5" data-line-number="5">  <span class="fu">description:</span><span class="at"> </span><span class="st">&#39;&#39;</span></a>
<a class="sourceLine" id="cb26-6" data-line-number="6">  <span class="fu">trigger:</span></a>
<a class="sourceLine" id="cb26-7" data-line-number="7">  <span class="kw">-</span> <span class="fu">platform:</span><span class="at"> template</span></a>
<a class="sourceLine" id="cb26-8" data-line-number="8">    <span class="fu">value_template:</span><span class="at"> </span><span class="st">&#39;{{state_attr(&#39;</span><span class="er">&#39;climate.woonkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) &lt; state_attr(&#39;&#39;climate.woonkamer&#39;&#39;,</span></a>
<a class="sourceLine" id="cb26-9" data-line-number="9">      <span class="st">&#39;&#39;</span>current_temperature<span class="st">&#39;&#39;</span>)}}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-10" data-line-number="10"><span class="st">  - platform: time_pattern</span></a>
<a class="sourceLine" id="cb26-11" data-line-number="11"><span class="st">    minutes: &#39;</span>2<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-12" data-line-number="12"><span class="st">  condition:</span></a>
<a class="sourceLine" id="cb26-13" data-line-number="13"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb26-14" data-line-number="14"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb26-15" data-line-number="15"><span class="st">    state: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-16" data-line-number="16"><span class="st">  - condition: template</span></a>
<a class="sourceLine" id="cb26-17" data-line-number="17"><span class="st">    value_template: &#39;</span><span class="kw">{</span>{state_attr(&#39;&#39;climate.woonkamer&#39;&#39;, &#39;&#39;temperature&#39;&#39;) &lt; state_attr(&#39;&#39;climate.woonkamer&#39;&#39;,</a>
<a class="sourceLine" id="cb26-18" data-line-number="18">      &#39;&#39;current_temperature&#39;&#39;)<span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-19" data-line-number="19"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-20" data-line-number="20"><span class="st">  - service: climate.turn_off</span></a>
<a class="sourceLine" id="cb26-21" data-line-number="21"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-22" data-line-number="22"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb26-23" data-line-number="23"><span class="st">  mode: restart</span></a>
<a class="sourceLine" id="cb26-24" data-line-number="24"><span class="st">  max: 10</span></a>
<a class="sourceLine" id="cb26-25" data-line-number="25"></a>
<a class="sourceLine" id="cb26-26" data-line-number="26"><span class="st">- id: &#39;</span>1608290218329<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-27" data-line-number="27"><span class="st">  alias: Bij opstaan aanwezigheid uit</span></a>
<a class="sourceLine" id="cb26-28" data-line-number="28"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-29" data-line-number="29"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-30" data-line-number="30"><span class="st">  - platform: time</span></a>
<a class="sourceLine" id="cb26-31" data-line-number="31"><span class="st">    at: input_datetime.opstaan</span></a>
<a class="sourceLine" id="cb26-32" data-line-number="32"><span class="st">  condition: []</span></a>
<a class="sourceLine" id="cb26-33" data-line-number="33"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-34" data-line-number="34"><span class="st">  - service: input_boolean.turn_off</span></a>
<a class="sourceLine" id="cb26-35" data-line-number="35"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-36" data-line-number="36"><span class="st">    entity_id: input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb26-37" data-line-number="37"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-38" data-line-number="38"><span class="st">- id: &#39;</span>1608297641472<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-39" data-line-number="39"><span class="st">  alias: terug naar instelwaarde bij restart</span></a>
<a class="sourceLine" id="cb26-40" data-line-number="40"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-41" data-line-number="41"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-42" data-line-number="42"><span class="st">  - platform: time_pattern</span></a>
<a class="sourceLine" id="cb26-43" data-line-number="43"><span class="st">    minutes: &#39;</span>15<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-44" data-line-number="44"><span class="st">  condition: []</span></a>
<a class="sourceLine" id="cb26-45" data-line-number="45"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-46" data-line-number="46"><span class="st">  - service: climate.set_temperature</span></a>
<a class="sourceLine" id="cb26-47" data-line-number="47"><span class="st">    data:</span></a>
<a class="sourceLine" id="cb26-48" data-line-number="48"><span class="st">      temperature: &#39;</span><span class="kw">{</span>{states(&#39;&#39;input_number.current_insteltemp_slaapkamer&#39;&#39;)<span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-49" data-line-number="49"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb26-50" data-line-number="50"><span class="st">  - service: climate.set_temperature</span></a>
<a class="sourceLine" id="cb26-51" data-line-number="51"><span class="st">    data:</span></a>
<a class="sourceLine" id="cb26-52" data-line-number="52"><span class="st">      temperature: &#39;</span><span class="kw">{</span>{states(&#39;&#39;input_number.current_insteltemp_woonkamer&#39;&#39;)<span class="kw">}</span>}<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-53" data-line-number="53"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb26-54" data-line-number="54"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-55" data-line-number="55"><span class="st">- id: &#39;</span>1608298248187<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-56" data-line-number="56"><span class="st">  alias: Helper schakelaars voor klimaat</span></a>
<a class="sourceLine" id="cb26-57" data-line-number="57"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-58" data-line-number="58"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-59" data-line-number="59"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-60" data-line-number="60"><span class="st">    entity_id: input_boolean.schakelaar_slaapkamer</span></a>
<a class="sourceLine" id="cb26-61" data-line-number="61"><span class="st">    to: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-62" data-line-number="62"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-63" data-line-number="63"><span class="st">    entity_id: input_boolean.schakelaar_woonkamer</span></a>
<a class="sourceLine" id="cb26-64" data-line-number="64"><span class="st">    to: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-65" data-line-number="65"><span class="st">  condition: []</span></a>
<a class="sourceLine" id="cb26-66" data-line-number="66"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-67" data-line-number="67"><span class="st">  - service: switch.turn_on</span></a>
<a class="sourceLine" id="cb26-68" data-line-number="68"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-69" data-line-number="69"><span class="st">    entity_id: switch.relay</span></a>
<a class="sourceLine" id="cb26-70" data-line-number="70"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-71" data-line-number="71"><span class="st">- id: &#39;</span>1608298257006<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-72" data-line-number="72"><span class="st">  alias: Helper schakelaars UIT klimaat</span></a>
<a class="sourceLine" id="cb26-73" data-line-number="73"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-74" data-line-number="74"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-75" data-line-number="75"><span class="st">  - platform: time_pattern</span></a>
<a class="sourceLine" id="cb26-76" data-line-number="76"><span class="st">    seconds: &#39;</span>5<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-77" data-line-number="77"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-78" data-line-number="78"><span class="st">    entity_id: input_boolean.schakelaar_slaapkamer</span></a>
<a class="sourceLine" id="cb26-79" data-line-number="79"><span class="st">    to: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-80" data-line-number="80"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-81" data-line-number="81"><span class="st">    entity_id: input_boolean.schakelaar_woonkamer</span></a>
<a class="sourceLine" id="cb26-82" data-line-number="82"><span class="st">    to: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-83" data-line-number="83"><span class="st">  condition:</span></a>
<a class="sourceLine" id="cb26-84" data-line-number="84"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb26-85" data-line-number="85"><span class="st">    entity_id: input_boolean.schakelaar_slaapkamer</span></a>
<a class="sourceLine" id="cb26-86" data-line-number="86"><span class="st">    state: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-87" data-line-number="87"><span class="st">  - condition: state</span></a>
<a class="sourceLine" id="cb26-88" data-line-number="88"><span class="st">    entity_id: input_boolean.schakelaar_woonkamer</span></a>
<a class="sourceLine" id="cb26-89" data-line-number="89"><span class="st">    state: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-90" data-line-number="90"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-91" data-line-number="91"><span class="st">  - service: switch.turn_off</span></a>
<a class="sourceLine" id="cb26-92" data-line-number="92"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-93" data-line-number="93"><span class="st">    entity_id: switch.relay</span></a>
<a class="sourceLine" id="cb26-94" data-line-number="94"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-95" data-line-number="95"><span class="st">- id: &#39;</span>1608318082068<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-96" data-line-number="96"><span class="st">  alias: Aanwezigheid aan</span></a>
<a class="sourceLine" id="cb26-97" data-line-number="97"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-98" data-line-number="98"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-99" data-line-number="99"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-100" data-line-number="100"><span class="st">    entity_id: input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb26-101" data-line-number="101"><span class="st">    to: &#39;</span>on<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-102" data-line-number="102"><span class="st">  condition: []</span></a>
<a class="sourceLine" id="cb26-103" data-line-number="103"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-104" data-line-number="104"><span class="st">  - service: climate.turn_on</span></a>
<a class="sourceLine" id="cb26-105" data-line-number="105"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-106" data-line-number="106"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb26-107" data-line-number="107"><span class="st">  - service: climate.turn_on</span></a>
<a class="sourceLine" id="cb26-108" data-line-number="108"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-109" data-line-number="109"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb26-110" data-line-number="110"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-111" data-line-number="111"><span class="st">- id: &#39;</span>1608318174333<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-112" data-line-number="112"><span class="st">  alias: Aanwezigheid uit</span></a>
<a class="sourceLine" id="cb26-113" data-line-number="113"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-114" data-line-number="114"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-115" data-line-number="115"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-116" data-line-number="116"><span class="st">    entity_id: input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb26-117" data-line-number="117"><span class="st">    to: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-118" data-line-number="118"><span class="st">  condition: []</span></a>
<a class="sourceLine" id="cb26-119" data-line-number="119"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-120" data-line-number="120"><span class="st">  - service: climate.turn_off</span></a>
<a class="sourceLine" id="cb26-121" data-line-number="121"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-122" data-line-number="122"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb26-123" data-line-number="123"><span class="st">  - service: climate.turn_off</span></a>
<a class="sourceLine" id="cb26-124" data-line-number="124"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-125" data-line-number="125"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb26-126" data-line-number="126"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-127" data-line-number="127"><span class="st">- id: &#39;</span>1608318195860<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-128" data-line-number="128"><span class="st">  alias: Aanwezigheid uit</span></a>
<a class="sourceLine" id="cb26-129" data-line-number="129"><span class="st">  description: &#39;&#39;</span></a>
<a class="sourceLine" id="cb26-130" data-line-number="130"><span class="st">  trigger:</span></a>
<a class="sourceLine" id="cb26-131" data-line-number="131"><span class="st">  - platform: state</span></a>
<a class="sourceLine" id="cb26-132" data-line-number="132"><span class="st">    entity_id: input_boolean.iemandthuis</span></a>
<a class="sourceLine" id="cb26-133" data-line-number="133"><span class="st">    to: &#39;</span>off<span class="st">&#39;</span></a>
<a class="sourceLine" id="cb26-134" data-line-number="134"><span class="st">  condition: []</span></a>
<a class="sourceLine" id="cb26-135" data-line-number="135"><span class="st">  action:</span></a>
<a class="sourceLine" id="cb26-136" data-line-number="136"><span class="st">  - service: climate.turn_off</span></a>
<a class="sourceLine" id="cb26-137" data-line-number="137"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-138" data-line-number="138"><span class="st">    entity_id: climate.slaapkamer</span></a>
<a class="sourceLine" id="cb26-139" data-line-number="139"><span class="st">  - service: climate.turn_off</span></a>
<a class="sourceLine" id="cb26-140" data-line-number="140"><span class="st">    data: {}</span></a>
<a class="sourceLine" id="cb26-141" data-line-number="141"><span class="st">    entity_id: climate.woonkamer</span></a>
<a class="sourceLine" id="cb26-142" data-line-number="142"><span class="st">  mode: single</span></a>
<a class="sourceLine" id="cb26-143" data-line-number="143"><span class="st">{% endraw %}</span></a></code></pre></div>
<h2 id="links-to-other-similar"><span class="header-section-number">0.4</span> Links to other similar</h2>
<p>I used these webpages for inspiration:</p>
<p><a href="https://siytek.com/the-ultimate-home-assistant-diy-thermostat-guide-for-single-or-multiple-zone-heating/">Siytek - the ultimate home-assistant DIY guide for multiple zone heating</a></p>
<p><a href="https://www.youtube.com/watch?v=LlPHrdXHBTU">Hacking a Eqiva EQ-3 thermostatic radiator valve.</a></p>
<p>An interesting idea: <a href="https://www.youtube.com/watch?v=-beIaL-RmvQ">using thermal sensors as room presence detection</a></p>
