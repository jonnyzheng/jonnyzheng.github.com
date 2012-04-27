--- 
categories: 
  - "Ruby &amp; Rails"
comments: true
layout: post
published: true
status: publish
tags: []
title: "根据IP获得城市名称 Ruby 版"
type: post
---
很多本地化生活网站的一个基本功能就是根据用户的IP地址判断所在的城市，基本的逻辑就是有一个ip库，根据ip库里的记录找到用户ip地址所在的范围。

IP库没有太多选择，我在网上找到的一个<a href="http://www.maxmind.com/app/ip-location">http://www.maxmind.com/app/ip-location</a>， 这个网站本身提供收费的IP库和免费的IP库，收费的库在准确率上要稍微高一些，拿城市的库做一个例子，左边是免费版，右边是收费版：
<table><tbody>
<tr>
<th></th>
<td>GeoLite City</td>
<td><span style="color: #000000;"><a href="http://www.maxmind.com/app/city"><span style="color: #000000;">GeoIP City</span></a></span></td>
</tr>
<tr>
<td>Cost</td>
<td>Free</td>
<td>$370 initial, $90 per month updates</td>
</tr>
<tr>
<td>Coverage</td>
<td>Worldwide</td>
<td>Worldwide</td>
</tr>
<tr>
<td>Accuracy</td>
<td>Over 99.5% on a country level and 79% on a city level for the US within a 25 mile radius. <a href="http://www.maxmind.com/app/geolite_city_accuracy">More details</a>
</td>
<td>Over 99.8% on a country level and 83% on a city level for the US within a 25 mile radius. <a href="http://www.maxmind.com/app/city_accuracy">More details</a>
</td>
</tr>
<tr>
<td>Redistribution</td>
<td>Free, subject to GPL/LGPL for APIs and <a href="http://geolite.maxmind.com/download/geoip/database/LICENSE.txt">database license</a>. <a href="http://www.maxmind.com/app/builder">Commercial redistribution licenses</a> are available</td>
<td>Please contact us.</td>
</tr>
<tr>
<td>Updates</td>
<td>Updated monthly, at the beginning of each month</td>
<td>Updated monthly. For binary format, weekly updates, automated updates available by using geoipupdate program included with<a href="http://www.maxmind.com/app/c">C API</a>
</td>
</tr>
</tbody></table>
ruby 的使用很简单，有一个gem库: <a href="http://geoip.rubyforge.org/">http://geoip.rubyforge.org/</a>
[ccw_ruby]
require ‘geoip’
c = GeoIP.new(‘GeoIP.dat’).country(‘www.nokia.com’)
=> [“www.nokia.com”, “147.243.3.83”, 69, “FI”, “FIN”, “Finland”, “EU”]
c.country_code3
=> “FIN”
c.to_hash
=> {:country_code3=>"FIN", :country_name=>"Finland", :continent_code=>"EU",
:request=>"www.nokia.com", :country_code=>69, :country_code2=>"FI", :ip=>"147.243.3.83"}


c = GeoIP.new(‘GeoLiteCity.dat’).city(‘github.com’)
=> [“github.com”, “207.97.227.239”, “US”, “USA”, “United States”, “NA”, “CA”,
“San Francisco”, “94110”, 37.7484, -122.4156, 807, 415, “America/Los_Angeles”]
>> c.longitude
=> -122.4156
>> c.timezone
=> “America/Los_Angeles”


c = GeoIP.new(‘GeoIPASNum.dat’).asn(“www.fsb.ru”)
=> [“AS8342”, “RTComm.RU Autonomous System”]
[/ccw_ruby]

我下了一个citylite的免费库，只试了我自己的地址，判断是准确的，还没有找其他的地方进行测试,79%的准确率不高也不低，用作预判断还是可以接受的。
