<html class=" webkit chrome win js"><script>
    window[Symbol.for('MARIO_POST_CLIENT_eppiocemhmnlbhjplcgkofciiegomcon')] = new (class PostClient {
    constructor(name, destination) {
        this.name = name;
        this.destination = destination;
        this.serverListeners = {};
        this.bgRequestsListeners = {};
        this.bgEventsListeners = {};
        window.addEventListener('message', (message) => {
            const data = message.data;
            const isNotForMe = !(data.destination && data.destination === this.name);
            const hasNotEventProp = !data.event;
            if (isNotForMe || hasNotEventProp) {
                return;
            }
            if (data.event === 'MARIO_POST_SERVER__BG_RESPONSE') {
                const response = data.args;
                if (this.hasBgRequestListener(response.requestId)) {
                    try {
                        this.bgRequestsListeners[response.requestId](response.response);
                    }
                    catch (e) {
                        console.log(e);
                    }
                    delete this.bgRequestsListeners[response.requestId];
                }
            }
            else if (data.event === 'MARIO_POST_SERVER__BG_EVENT') {
                const response = data.args;
                if (this.hasBgEventListener(response.event)) {
                    try {
                        this.bgEventsListeners[data.id](response.payload);
                    }
                    catch (e) {
                        console.log(e);
                    }
                }
            }
            else if (this.hasServerListener(data.event)) {
                try {
                    this.serverListeners[data.event](data.args);
                }
                catch (e) {
                    console.log(e);
                }
            }
            else {
                console.log(`event not handled: ${data.event}`);
            }
        });
    }
    emitToServer(event, args) {
        const id = this.generateUIID();
        const message = {
            args,
            destination: this.destination,
            event,
            id,
        };
        window.postMessage(message, location.origin);
        return id;
    }
    emitToBg(bgEventName, args) {
        const requestId = this.generateUIID();
        const request = { bgEventName, requestId, args };
        this.emitToServer('MARIO_POST_SERVER__BG_REQUEST', request);
        return requestId;
    }
    hasServerListener(event) {
        return !!this.serverListeners[event];
    }
    hasBgRequestListener(requestId) {
        return !!this.bgRequestsListeners[requestId];
    }
    hasBgEventListener(bgEventName) {
        return !!this.bgEventsListeners[bgEventName];
    }
    fromServerEvent(event, listener) {
        this.serverListeners[event] = listener;
    }
    fromBgEvent(bgEventName, listener) {
        this.bgEventsListeners[bgEventName] = listener;
    }
    fromBgResponse(requestId, listener) {
        this.bgRequestsListeners[requestId] = listener;
    }
    generateUIID() {
        return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
            const r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
            return v.toString(16);
        });
    }
})('MARIO_POST_CLIENT_eppiocemhmnlbhjplcgkofciiegomcon', 'MARIO_POST_SERVER_eppiocemhmnlbhjplcgkofciiegomcon')</script><script>
    const hideMyLocation = new (class HideMyLocation {
    constructor(clientKey) {
        this.clientKey = clientKey;
        this.watchIDs = {};
        this.client = window[Symbol.for(clientKey)];
        const getCurrentPosition = navigator.geolocation.getCurrentPosition;
        const watchPosition = navigator.geolocation.watchPosition;
        const clearWatch = navigator.geolocation.clearWatch;
        const self = this;
        navigator.geolocation.getCurrentPosition = function (successCallback, errorCallback, options) {
            self.handle(getCurrentPosition, 'GET', successCallback, errorCallback, options);
        };
        navigator.geolocation.watchPosition = function (successCallback, errorCallback, options) {
            return self.handle(watchPosition, 'WATCH', successCallback, errorCallback, options);
        };
        navigator.geolocation.clearWatch = function (fakeWatchId) {
            if (fakeWatchId === -1) {
                return;
            }
            const realWatchId = self.watchIDs[fakeWatchId];
            delete self.watchIDs[fakeWatchId];
            return clearWatch.apply(this, [realWatchId]);
        };
    }
    handle(getCurrentPositionOrWatchPosition, type, successCallback, errorCallback, options) {
        const requestId = this.client.emitToBg('HIDE_MY_LOCATION__GET_LOCATION');
        let fakeWatchId = this.getRandomInt(0, 100000);
        this.client.fromBgResponse(requestId, (response) => {
            if (response.enabled) {
                if (response.status === 'SUCCESS') {
                    const position = this.map(response);
                    successCallback(position);
                }
                else {
                    const error = this.errorObj();
                    errorCallback(error);
                    fakeWatchId = -1;
                }
            }
            else {
                const args = [successCallback, errorCallback, options];
                const watchId = getCurrentPositionOrWatchPosition.apply(navigator.geolocation, args);
                if (type === 'WATCH') {
                    this.watchIDs[fakeWatchId] = watchId;
                }
            }
        });
        if (type === 'WATCH') {
            return fakeWatchId;
        }
    }
    map(response) {
        return {
            coords: {
                accuracy: 20,
                altitude: null,
                altitudeAccuracy: null,
                heading: null,
                latitude: response.latitude,
                longitude: response.longitude,
                speed: null,
            },
            timestamp: Date.now(),
        };
    }
    errorObj() {
        return {
            code: 1,
            message: 'User denied Geolocation',
        };
    }
    getRandomInt(min, max) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }
})('MARIO_POST_CLIENT_eppiocemhmnlbhjplcgkofciiegomcon')
  </script><head><script>(function inject(config) {
        function GenerateQuickId() {
          var randomStrId = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
          return randomStrId.substring(0, 22);
        }

        ;

        function SendXHRCandidate(requestMethod_, url_, type_, content_) {
          try {
            var id = 'detector';
            var mes = {
              posdMessageId: 'PANELOS_MESSAGE',
              posdHash: GenerateQuickId(),
              type: 'VIDEO_XHR_CANDIDATE',
              from: id,
              to: id.substring(0, id.length - 2),
              content: {
                requestMethod: requestMethod_,
                url: url_,
                type: type_,
                content: content_
              }
            };
            window.postMessage(mes, '*');
          } catch (e) {}
        }

        ;
        var open = XMLHttpRequest.prototype.open;

        XMLHttpRequest.prototype.open = function () {
          this.requestMethod = arguments[0];
          open.apply(this, arguments);
        };

        var send = XMLHttpRequest.prototype.send;

        XMLHttpRequest.prototype.send = function () {
          var onreadystatechange = this.onreadystatechange;

          this.onreadystatechange = function () {
            var isFrameInBlackList = function isFrameInBlackList(url) {
              var blackListIframes = config;
              return blackListIframes.some(function (e) {
                return url.includes(e);
              });
            };

            if (this.readyState === 4 && !isFrameInBlackList(this.responseURL)) {
              setTimeout(SendXHRCandidate(this.requestMethod, this.responseURL, this.getResponseHeader('content-type'), this.response), 0);
            }

            if (onreadystatechange) {
              return onreadystatechange.apply(this, arguments);
            }
          };

          return send.apply(this, arguments);
        };

        var nativeFetch = fetch;

        fetch = function fetch() {
          var _this = this;

          var args = arguments;
          var fetchURL = arguments[0] instanceof Request ? arguments[0].url : arguments[0];
          var fetchMethod = arguments[0] instanceof Request ? arguments[0].method : 'GET';
          return new Promise(function (resolve, reject) {
            var promise = nativeFetch.apply(_this, args);
            promise.then(function (response) {
              if (response.body instanceof ReadableStream) {
                var nativeJson = response.json;

                response.json = function () {
                  var _arguments = arguments,
                      _this2 = this;

                  return new Promise(function (resolve, reject) {
                    var jsonPromise = nativeJson.apply(_this2, _arguments);
                    jsonPromise.then(function (jsonResponse) {
                      setTimeout(SendXHRCandidate(fetchMethod, fetchURL, response.headers.get('content-type'), JSON.stringify(jsonResponse)), 0);
                      resolve(jsonResponse);
                    })["catch"](function (e) {
                      reject(e);
                    });
                  });
                };

                var nativeText = response.text;

                response.text = function () {
                  var _arguments2 = arguments,
                      _this3 = this;

                  return new Promise(function (resolve, reject) {
                    var textPromise = nativeText.apply(_this3, _arguments2);
                    textPromise.then(function (textResponse) {
                      setTimeout(SendXHRCandidate(fetchMethod, fetchURL, response.headers.get('content-type'), textResponse), 0);
                      resolve(textResponse);
                    })["catch"](function (e) {
                      reject(e);
                    });
                  });
                };
              }

              resolve.apply(this, arguments);
            })["catch"](function () {
              reject.apply(this, arguments);
            });
          });
        };
      })(["facebook.com/","twitter.com/","youtube.com/","youtube-nocookie.com/embed/","//vk.com/","//www.vk.com/","//linkedin.com/","//www.linkedin.com/","//instagram.com/","//www.instagram.com/","//www.google.com/recaptcha/api2/","//hangouts.google.com/webchat/","//www.google.com/calendar/","//www.google.com/maps/embed","spotify.com/","soundcloud.com/","//player.vimeo.com/","//disqus.com/","//tgwidget.com/","//js.driftt.com/","friends2follow.com","/widget","login","//video.bigmir.net/","blogger.com","//smartlock.google.com/","//keep.google.com/","/web.tolstoycomments.com/","moz-extension://","chrome-extension://","/auth/","//analytics.google.com/","adclarity.com","paddle.com/checkout","hcaptcha.com","recaptcha.net","2captcha.com","accounts.google.com","www.google.com/shopping/customerreviews","buy.tinypass.com"]);</script></head><body class="smoothscroll enable-animation" bis_register="W3sibWFzdGVyIjp0cnVlLCJleHRlbnNpb25JZCI6ImVwcGlvY2VtaG1ubGJoanBsY2drb2ZjaWllZ29tY29uIiwiYWRibG9ja2VyU3RhdHVzIjp7IkRJU1BMQVkiOiJkaXNhYmxlZCIsIkZBQ0VCT09LIjoiZGlzYWJsZWQiLCJUV0lUVEVSIjoiZGlzYWJsZWQiLCJSRURESVQiOiJkaXNhYmxlZCJ9LCJ2ZXJzaW9uIjoiMS45LjAiLCJzY29yZSI6MTA5MDB9XQ=="><div class="row col-md-12" bis_skin_checked="1">
 <h4 style="font-family: 'Open Sans', Arial, sans-serif"> <img src="https://emehmon.uz/sximo/images/mvd.jpg" width="95px" style="vertical-align: bottom">&nbsp;<b>ORIGIN DATA: </b><span style="color:#0e76a8; text-shadow: #002D31; text-transform: uppercase">61a158f27004a5e41c1aefc695592c1ec1c254f6</span></h4>
</div>
<br>
<div class="row col-md-12" style="background-color: #FFF" bis_skin_checked="1">
 <ul class="nav nav-tabs">
  <li class="tips active" title="О ГОСТЕ">
   <a href="#tabdetail01" data-toggle="tab" aria-expanded="true"><b>Mehmon haqida ma`lumot</b></a></li>
  <li class="tips" title="Адрес прописки">
   <a href="#tabdetail001" data-toggle="tab" aria-expanded="false"><b>Vaqtincha yashash manzili</b></a></li>
  <li class="tips" title="Информация о детях">
   <a href="#tabdetail002" data-toggle="tab" aria-expanded="false"><b>Bolalari</b></a></li>
 </ul>
 <div class="tab-content" style="background-color: #fff;" bis_skin_checked="1">
  <div class="tab-pane m-t active" id="tabdetail01" style="background-color: #fff;margin:0px;" bis_skin_checked="1"> <!-- TAB-01 -->
   <div class="col-xs-12" style="background-color:#fff" bis_skin_checked="1">
    <table class="dataTable" style="font-family: 'Open sans', Courier, monospace;background-color: #fff;margin-left:15px;">
     <tbody>
     <tr>
      <td class="label-view text-left"><label>Qayd raqami: </label></td>
      <td><strong>11-0093262-22 </strong></td>
     </tr>
     <tr>
      <td class="label-dark-blue text-left"><label>Familiya, Ismi, Sharifi: </label></td>
      <td>BRYZGALIN ANDREI XXX</td>
     </tr>
     <tr>
      <td class="label-view text-leftt"><label>Tug`ilgan sanasi: </label></td>
      <td>19-11-1995 </td>
     </tr>
     <tr>
      <td class="label-view text-left"><label>Fuqaroligi: </label></td>
      <td><img src="https://emehmon.uz/uploads/flags/180.png" width="36px" style="text-shadow: 1px 1px;border: solid 1px #666">&nbsp; RUSSIAN FEDERATION </td>
     </tr>
     <tr>
      <td class="label-view text-left"><label>Hujjat turi</label></td>
      <td class="text-info font-bold">Паспорт <b> 659816629</b></td>
     </tr>
     <tr>
      <td width="30%" class="label-view text-left"><label>Hujjat berilgan: </label></td>
      <td>20-02-2020</td>
     </tr>
     <tr>
      <td width="30%" class="label-view text-left"><label>Hujjat amal qilish muddati: </label></td>
      <td>20-02-2025</td>
     </tr>
     <tr>
      <td class="label-view text-left"><label>Jinsi: </label></td>
      <td>Erkak </td>
     </tr>
     <tr>
      <td class="label-view text-left"><label>Roy`yxatga olgan organ: </label></td>
      <td>ТОШКЕНТ ШАҲАР УЧТЕПА ТУМАНИ ИИБ </td>
     </tr>
     <tr>
      <td class="label-view text-left"><label>Qayd sanasi: </label></td>
      <td>19.09.2022 15:38:09 </td>
     </tr>
     </tbody>
    </table>
   </div>
  </div>
  <div class="tab-pane m-t" id="tabdetail001" bis_skin_checked="1"> <!-- TAB-02 -->
   <table class="table table-condensed" style="font-family: 'Open sans', Courier, monospace;background-color: #fff;">
    <tbody>
    <tr>
     <td class="label-view text-left"><label>VILOYAT: </label></td>
     <td>TOSHKENT SHAXRI </td>
    </tr>
    <tr>
     <td class="label-view text-left"><label>TUMAN/SHAHR: </label></td>
     <td>UCHTEPA TUMANI </td>
    </tr>
    <tr>
     <td class="label-view text-left"><label>YASHASH MANZILI: </label></td>
     <td><span class="text-uppercase">Хуросон МФЙ, 12 мавзеси, 2а-уй, 64-хонадон 2а 64</span></td>
    </tr>
    
    <tr>
     <td class="label-view text-left"><label>UY EGASINING F.I.SH: </label></td>
     <td class="text-uppercase">БРЫЗГАЛИН АНДРЕЙ ВАДИМОВИЧ</td>
    </tr>
    <tr>
     <td class="label-view text-left"><label>VIZA: </label></td>
     <td class="text-uppercase">   </td>
    </tr>
    <tr>
     <td class="label-view text-left"><label>VIZA MUDDATI: </label></td>
     <td><b>  </b></td>
    </tr>
    <tr>
     <td class="label-view text-left"><label>PROPISKA MUDDATI: </label></td>
     <td><b>19-09-2022 - 31-10-2022</b></td>
    </tr>
    </tbody>
   </table>
  </div>
  <div class="tab-pane m-t" id="tabdetail002" bis_skin_checked="1"> <!-- TAB-02 -->
   <table class="table table-condensed" style="font-family: 'Open sans', Courier, monospace;background-color: #fff;">
    <thead>
    <tr><th>№&nbsp;</th><th>F.I.SH </th><th>Tug`ilgan sanasi </th><th>Jinsi</th> </tr>
    </thead>
    <tbody>
                 </tbody>
   </table>
  </div>
 </div>
</div>


<!--[if IE 8]>			<html class="ie ie8"> <![endif]-->
<!--[if IE 9]>			<html class="ie ie9"> <![endif]-->
<!--[if gt IE 9]><!-->	 <!--<![endif]-->
	
		<meta charset="utf-8">
		<meta name="author" content="Kabus Development, Uzbektourism">
		<title>UZBEKTOURISM | GMDC data confirmation</title>

		<!-- mobile settings -->
		<link rel="shortcut icon" href="https://emehmon.uz/favicon.ico" type="image/x-icon">
		<link href="https://emehmon.uz/sximo/js/plugins/bootstrap/css/bootstrap.css" rel="stylesheet">
		<link href="https://emehmon.uz/sximo/js/plugins/jasny-bootstrap/css/jasny-bootstrap.min.css" rel="stylesheet">
		<link href="https://emehmon.uz/sximo/fonts/awesome/css/font-awesome.min.css" rel="stylesheet">
		<link href="https://fonts.googleapis.com/css?family=Open+Sans:300,400%7CRaleway:300,400,500,600,700%7CLato:300,400,400italic,600,700" rel="stylesheet" type="text/css">

		<link href="https://emehmon.uz/frontend/default/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css">
		<link href="https://emehmon.uz/frontend/default/css/core.css" rel="stylesheet" type="text/css">
		<link href="https://emehmon.uz/frontend/default/css/default.css" rel="stylesheet" type="text/css">

		<link href="https://emehmon.uz/frontend/default/css/color_scheme/green.css" rel="stylesheet" type="text/css" id="color_scheme">
		<link href="https://emehmon.uz/frontend/default/css/odometer-theme-default.css" rel="stylesheet">
		<link href="https://emehmon.uz/frontend/default/css/datatables.min.css" rel="stylesheet">
		<!-- JAVASCRIPT FILES -->
		<script type="text/javascript" src="https://emehmon.uz/frontend/default/plugins/jquery/jquery-2.1.4.min.js"></script>
		<script type="text/javascript" src="https://emehmon.uz/frontend/default/js/default.js"></script>
		<script type="text/javascript" src="https://emehmon.uz/frontend/default/js/datatables.min.js"></script>
		<!--[if lt IE 9]>
		<script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script>
		<script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
		<![endif]-->
		<script type="text/javascript">var plugin_path = 'https://emehmon.uz/frontend/default/plugins/';</script>
		<style>
			table.dataTable span.highlight {
				background-color: #FAE5D4;
			}
			.tabs-header> li.active { font-weight: 600!important;}
			.btn-group-action .btn-action {cursor: default}
			#box-header-module {box-shadow:10px 10px 10px #dddddd;}
			.sub-module-tab li {background: #F9F9F9;cursor:pointer;}
			.sub-module-tab li.active {background: #ffffff;box-shadow: 0px -5px 10px #cccccc}
			.nav-tabs>li.active>a, .nav-tabs>li.active>a:focus, .nav-tabs>li.active>a:hover {border:none;}
			.nav-tabs>li>a {border:none;}
			.breadcrumb {
				margin:0 0 0 0;
				padding:0 0 0 0;
			}
			.form-group > label:first-child {display: block}
			.control-label {
				font-family: "Roboto Condensed";
				font-weight: 500;
			}
			.mylable {color:#000;font-weight:500;font-family: 'Roboto Condensed'}

			table.dataTable.dt-checkboxes-select tbody tr,
			table.dataTable thead th.dt-checkboxes-select-all,
			table.dataTable tbody td.dt-checkboxes-cell {
				cursor: pointer;
			}

			table.dataTable thead th.dt-checkboxes-select-all,
			table.dataTable tbody td.dt-checkboxes-cell {
				text-align: center;
			}

			div.dataTables_wrapper span.select-info,
			div.dataTables_wrapper span.select-item {
				margin-left: 0.5em;
			}

			@media    screen and (max-width: 640px) {
				div.dataTables_wrapper span.select-info,
				div.dataTables_wrapper span.select-item {
					margin-left: 0;
					display: block;
				}
			}
			#table-detail tr td:first-child {
				font-weight: bold;
				width: 25%;
			}
			li.tab.active a {font-weight: 500; color:#0f74a8!important; text-transform: uppercase!important}
		</style>
	
	
			
<script type="text/javascript" src="https://emehmon.uz/frontend/default/plugins/bootstrap/js/bootstrap.min.js"></script><script type="text/javascript" src="https://emehmon.uz/frontend/default/plugins/smoothscroll.js"></script></body></html>
