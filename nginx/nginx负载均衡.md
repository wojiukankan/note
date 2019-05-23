- 分发服务器配置文件
    - `http {}`中添加
    ```
     upstream test{
        server 192.168.126.131:90 weight=1;#从服务器1 ip:端口 权重
        server 192.168.126.129:90 weight=1;#从服务器2 ip:端口 权重
        server 192.168.126.128:90 weight=1;#从服务器3 ip:端口 权重
    }
    ```
    - `test`为自定义名称
    - `server{}`中的`location`改为,将请求代理到刚刚配置的test
    ```
        location / {
            proxy_pass http://test;
        }
    ```
- 从服务器配置文件,与`test`中的配置匹配
    ```
    server {
        listen 90;
        server_name 192.168.126.131;
            ...
    ```
- 分别重启各主从服务器的nginx
- ab压测结果如下
- 从服务器
    <table border=0 cellpadding=0 cellspacing=0 width=507 style='border-collapse:
     collapse;table-layout:fixed;width:381pt'>
     <col width=102 style='mso-width-source:userset;mso-width-alt:3264;width:77pt'>
     <col width=75 style='mso-width-source:userset;mso-width-alt:2400;width:56pt'>
     <col width=82 style='mso-width-source:userset;mso-width-alt:2624;width:62pt'>
     <col width=72 style='width:54pt'>
     <col width=80 style='mso-width-source:userset;mso-width-alt:2560;width:60pt'>
     <col width=96 style='mso-width-source:userset;mso-width-alt:3072;width:72pt'>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td colspan=6 height=20 class=xl68 width=507 style='height:15.0pt;width:381pt'>Connection
      Times (ms)</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'></td>
      <td class=xl65>min</td>
      <td colspan=2 class=xl65>mean[+/-sd]</td>
      <td class=xl65>median</td>
      <td class=xl65>max</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Connect:</td>
      <td class=xl65>0</td>
      <td class=xl65>1</td>
      <td class=xl65>0.6</td>
      <td class=xl65>1</td>
      <td class=xl65>7</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Processing:</td>
      <td class=xl65>426</td>
      <td class=xl65>3120</td>
      <td class=xl65>1719.2</td>
      <td class=xl65>3074</td>
      <td class=xl65>7062</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Waiting:</td>
      <td class=xl65>425</td>
      <td class=xl65>3119</td>
      <td class=xl65>1719.3</td>
      <td class=xl65>3074</td>
      <td class=xl65>7062</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Total:</td>
      <td class=xl65>427</td>
      <td class=xl65>3121</td>
      <td class=xl65>1719.1</td>
      <td class=xl65>3075</td>
      <td class=xl65>7063</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td colspan=6 height=18 class=xl65 width=144 style='height:13.5pt;width:108pt'>Percentage
      of the requests served within a certain time (ms)</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>50%</td>
      <td class=xl65>3075</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>66%</td>
      <td class=xl65>4074</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>75%</td>
      <td class=xl65>4649</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>80%</td>
      <td class=xl65>4868</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>90%</td>
      <td class=xl65>5515</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>95%</td>
      <td class=xl65>5798</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>98%</td>
      <td class=xl65>5947</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>99%</td>
      <td class=xl65>5981</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>100%</td>
      <td class=xl65>7063</td>
     </tr>
    </table>
- 主服务器
    <table border=0 cellpadding=0 cellspacing=0 width=507 style='border-collapse:
     collapse;table-layout:fixed;width:381pt'>
     <col width=102 style='mso-width-source:userset;mso-width-alt:3264;width:77pt'>
     <col width=75 style='mso-width-source:userset;mso-width-alt:2400;width:56pt'>
     <col width=82 style='mso-width-source:userset;mso-width-alt:2624;width:62pt'>
     <col width=72 style='width:54pt'>
     <col width=80 style='mso-width-source:userset;mso-width-alt:2560;width:60pt'>
     <col width=96 style='mso-width-source:userset;mso-width-alt:3072;width:72pt'>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td colspan=6 height=20 class=xl67 width=507 style='height:15.0pt;width:381pt'>Connection
      Times (ms)</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'></td>
      <td class=xl65>min</td>
      <td colspan=2 class=xl65>mean[+/-sd]</td>
      <td class=xl65>median</td>
      <td class=xl65>max</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Connect:</td>
      <td class=xl65>0</td>
      <td class=xl65>1</td>
      <td class=xl65>1</td>
      <td class=xl65>1</td>
      <td class=xl65>8</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Processing:</td>
      <td class=xl65>46</td>
      <td class=xl65>864</td>
      <td class=xl65>363.6</td>
      <td class=xl65>788</td>
      <td class=xl65>1597</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Waiting:</td>
      <td class=xl65>27</td>
      <td class=xl65>861</td>
      <td class=xl65>366.9</td>
      <td class=xl65>787</td>
      <td class=xl65>1597</td>
     </tr>
     <tr height=20 style='mso-height-source:userset;height:15.0pt'>
      <td height=20 class=xl65 style='height:15.0pt'>Total:</td>
      <td class=xl65>47</td>
      <td class=xl65>865</td>
      <td class=xl65>363.6</td>
      <td class=xl65>788</td>
      <td class=xl65>1598</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td colspan=6 height=18 class=xl66 width=144 style='height:13.5pt;width:108pt'>Percentage
      of the requests served within a certain time (ms)</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>50%</td>
      <td class=xl65>788</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>66%</td>
      <td class=xl65>1030</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>75%</td>
      <td class=xl65>1160</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>80%</td>
      <td class=xl65>1234</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>90%</td>
      <td class=xl65>1397</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>95%</td>
      <td class=xl65>1516</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>98%</td>
      <td class=xl65>1574</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>99%</td>
      <td class=xl65>1592</td>
     </tr>
     <tr height=18 style='height:13.5pt'>
      <td height=18 class=xl66 style='height:13.5pt'>100%</td>
      <td class=xl65>1598</td>
     </tr>
    </table>
- 500并发500请求情况下，负载均衡的访问速度明显提示
