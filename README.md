# weapp-MeetingAssistance
微信小程序-基于微信场景的会面小助手应用
<li>解读每个功能模块的实现过程，并讲解一下重要部分的代码。</li>
</ul>
<ol>
 	<li>小程序首页(/pages/first/first)：</li>
</ol>
小程序首页显示用户当前的会面情况。在小程序开启时，会调用App中的onLaunch方法，在其中会先调用wx.checkSession方法检测登陆态，如果登陆态有效，则上传openId到服务器，从数据库中读取属于该用户的会面，并回传到小程序本身，以列表形式显示出来。如果登陆态无效，则使用wx.login登陆换取sessionKey和openId，再从服务器数据库读取用户的会面情况。对于会面列表，可以通过点击查看该次会面的详情，此时会跳转到详情页面，即/pages/detail/detail。除了查看之外，可以点击发起会面按钮，此时会跳转到发起会面的页面，即/pages/makemeeting/makemeeting，其中填写会面信息即可实现发送。

2. 发起会面(/pages/makemeeting/makemeeting)：

该页面要求选择会面发生的时间，包括日期和时分，这部分使用picker实现选择的功能。在这之后是会面地点。会面地点可以通过地图选择，选择的地点对于接受方而言同样可以通过打开地图来显示。选择地点的时候会调用wx.chooseLocation接口在腾讯地图上选择会面的地点，选择成功后会将地点的名称、地址以及经纬度输出到小程序中，并结合会面时间、输入的会面描述一起通过request请求上传到服务器，进而存储到数据库中。会面地点也可以手动输入，此时在接受方将无法直接通过地图查看地理位置，一般适合输入双方都知晓的却又不在地图上有标记的地址。在信息输入完毕后，点击确定按钮，再通过界面右上角的转发按钮即可将会面转发到群组中。这里有一点需要说明。由于对于转发到个人，微信不支持返回转发的信息，因此只有转发到群组才可以通过shareTickets收到加密数据，并进而解密出openGId，对用户的身份，所属的群组进行识别，实现群组定向发送会面邀请的功能。

3.详情页面(/pages/detail/detail)：

在此页面会显示会面的详细信息，如会面日期，时分，地点及会面情况。对于通过地图设置的会面地点而言，用户在详情页面可以点击查看地点按钮查看具体的位置情况，此时会根据地点的经纬度调用wx.openLocation接口在地图上显现出会面的地点，使得用户可以对会面的地点有一个进一步的了解。而对于会面发起者手动输入的会面地点，一般来说这类地点为会面双方都知晓的地点，同时在腾讯地图上可能没有记录，因而无法通过查看地点按钮打开地图查看。除了查看会面详细信息之外，该页面还有一个设置会面提醒时间的按钮，通过点击该按钮会切换到选择提醒时间页面(/pages/remindtime/remindtime)，进而对提醒时间进行选择。

4.设置提醒时间页面(/pages/remindtime/remindtime)：

此页面有多个switch开关，每个开关代表一种提醒时间，可以设置不提醒、会面开始前5分钟提醒、会面开始前两小时提醒等情况。开关与开关之间存在着互斥的机制，同一时间最多只有一个按钮处于开启的状态。在挑选好提醒时间后，点击确定之后会回到详情页面，此时再点击确定按钮，会将会面提醒时间发送到服务器，服务器端收到了提醒时间后，会调用Java的Timer类绑定一个定时处理的任务TimerTask，在设定的时间到来时执行task任务。而task任务负责向微信服务器发送请求以实现发送模板消息到用户手机上的功能。

5.接收页面(/pages/accept/accept)：

这个页面只有会面的接收方能进入。接收方在群组收到发起者转发的会面邀请后，点击转发的卡片进入小程序，此时便会进入accept页面。页面中与detail页面一样存在着会面的详细信息，了解了会面的详情之后，可以通过点击接受或拒绝按钮来对该邀请进行处理。接受时会发送Post请求到服务器上，将该会面加入到该用户的数据库表中，以便之后查看以及设置提醒。拒绝则直接跳转到首页。
