<!--<!doctype html>
Copyright 2017-2020 JellyWare Inc. All Rights Reserved.
-->
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="description" content="BlueJelly">
    <!--<meta name="viewport" content="width=640, maximum-scale=1.0, user-scalable=yes">--＞
    <title>BlueJelly-ESP32  BLE DEMO</title>
    <link href="https://fonts.googleapis.com/css?family=Lato:100,300,400,700,900" rel="stylesheet" type="text/css">
    <link rel="stylesheet" href="style.css">
    <script type="text/javascript" src="bluejelly.js"></script>
    <script type="text/javascript" src="./smoothie.js"></script>
  </head>

<body>
<!--<div class="container">-->

    <!--<div class="title margin">-->
    
        <font color="orange" size='5'><p id="title"><strong>BlueJelly-ESP32  BLE DEMO</strong></p></font>
   

    <!--<div class="contents margin">-->
    
        <button id="startNotifications" class="button">Start Notify</button>
        <button id="stopNotifications" class="button">Stop Notify</button>
        <button id="write1" class="button">4方向</button>
        <button id="write2" class="button">8方向</button>
        <hr>
        <div id="svg">GRAPH AREA</div>
        <hr>
        <span id="data_text"> </span>
        <span>　　　　</span>
        <span id="data_text2"> </span>
        
        <!--<div id="device_name"> </div>
        <div id="uuid_name"> </div>
        
        <div id="status"> </div>-->

    
    <!--<div class="footer margin">
                For more information, see <a href="https://jellyware.jp/kurage" target="_blank">jellyware.jp</a> and <a href="https://github.com/electricbaka/bluejelly" target="_blank">GitHub</a> !
    </div>-->



<script>



let startflag=0;


//--------------------------------------------------
//Global変数
//--------------------------------------------------
//BlueJellyのインスタンス生成
const ble = new BlueJelly();
const ble2 = new BlueJelly();



//TimeSeriesのインスタンス生成
const ble_data = new TimeSeries();



//-------------------------------------------------
//smoothie.js
//-------------------------------------------------
function createTimeline() {
    const chart = new SmoothieChart({
        millisPerPixel: 20,
        grid: {
            fillStyle: '#ff8319',
            strokeStyle: '#ffffff',
            millisPerLine: 800
        },
        maxValue: 5000,
        minValue: 0
    });
    chart.addTimeSeries(ble_data, {
        strokeStyle: 'rgba(255, 255, 255, 1)',
        fillStyle: 'rgba(255, 255, 255, 0.2)',
        lineWidth: 4
    });
    chart.streamTo(document.getElementById("chart"), 500);
}


//--------------------------------------------------
//ロード時の処理
//--------------------------------------------------
window.onload = function () {
  //UUIDの設定
  ble.setUUID("UUID1","dd5f7232-1560-4792-953d-0b2015f15340","8796fa1b-986d-419a-8f84-137710a2354f");//TX　　Service UUID,Characteristic UUID
  ble2.setUUID("UUID1","dd5f7232-1560-4792-953d-0b2015f15340","1e630bfc-08ca-44c0-a7c5-58dae380884d");//RX　　Service UUID,Characteristic UUID
  
  
  //UUIDの取得方法→Powershellで　[Guid]::NewGuid()　と打つとUUIDが発行される。3回やって、1個目をService UUIDにして2個目と3個目をCharacteristic UUIDにする。
  //smoothie.js
  //createTimeline();
  
  
  main();

}


//--------------------------------------------------
//Scan後の処理
//--------------------------------------------------
ble.onScan = function (deviceName) {
  //document.getElementById('device_name').innerHTML = deviceName;
  document.getElementById('status').innerHTML = "found device!";
}


//--------------------------------------------------
//ConnectGATT後の処理
//--------------------------------------------------
ble.onConnectGATT = function (uuid) {
  console.log('> connected GATT!');

  //document.getElementById('uuid_name').innerHTML = uuid;
  //document.getElementById('status').innerHTML = "connected GATT!";
}


//--------------------------------------------------
//Sensor1 Read後の処理：得られたデータの表示など行う
//--------------------------------------------------
ble.onRead = function (data, uuid){
  //フォーマットに従って値を取得
  let value = "";
  for(let i = 0; i < data.byteLength; i++){
    value = value + String.fromCharCode(data.getInt8(i));
  }
  
  var array_value =value.split(",");

  //コンソールに値を表示
  //console.log(array_value);
  

  //HTMLにデータを表示
  document.getElementById('data_text').innerHTML = Number(array_value[0]);
  document.getElementById('data_text2').innerHTML = Number(array_value[1]);
  sosimode=Number(array_value[1]);
  startflag = 1;
  if(pre_sosimode != sosimode) gesflag=1;
  //console.log(startflag);

}


//--------------------------------------------------
//Sensot1 Write後の処理
//--------------------------------------------------
ble2.onWrite = function(uuid){
  //document.getElementById('uuid_name').innerHTML = uuid;
  //document.getElementById('status').innerHTML = "written data"
}


//--------------------------------------------------
//Start Notify後の処理
//--------------------------------------------------
ble.onStartNotify = function(uuid){
  console.log('> Start Notify!');

  //document.getElementById('uuid_name').innerHTML = uuid;
  //document.getElementById('status').innerHTML = "started Notify";
}


//--------------------------------------------------
//Stop Notify後の処理
//--------------------------------------------------
ble.onStopNotify = function(uuid){
  console.log('> Stop Notify!');

  //document.getElementById('uuid_name').innerHTML = uuid;
  //document.getElementById('status').innerHTML = "stopped Notify";
}


//-------------------------------------------------
//ボタンが押された時のイベント登録
//--------------------------------------------------
document.getElementById('startNotifications').addEventListener('click', function() {
      ble.startNotify('UUID1');
});

document.getElementById('stopNotifications').addEventListener('click', function() {
      ble.stopNotify('UUID1');
});


document.getElementById('write1').addEventListener('click', function() {
  //フォーマットに従って値を変換
  const textEncoder = new TextEncoder();
  const text_data ="mode0";
  const text_data_encoded = textEncoder.encode(text_data + '\n');
  ble2.write('UUID1', text_data_encoded);
  gesflag=1;
  sosimode=0;
});

document.getElementById('write2').addEventListener('click', function() {
  //フォーマットに従って値を変換
  const textEncoder = new TextEncoder();
  const text_data = "mode1";
  const text_data_encoded = textEncoder.encode(text_data + '\n');
  ble2.write('UUID1', text_data_encoded);
  gesflag=1;
  sosimode=1;
});







async function main() {
	while(true){
	    await wait(10); //
	    Create_grapf();
    }
}



const wait = (ms) => {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms);
  });
};



let p_pos=0;
let sosimode=0;
let screen_w = 900;
let screen_h = 700;
let lineX=screen_w/2;
let lineY=screen_h/2;
let gesflag=0;
let gescyc=0;
let get_uart=0;
let display_text=""
let baisu=2;
let pre_sosimode=0;




function Create_grapf() {

	let i=0;
	let ii=0;
	var plot_color = new Array('red', 'blue', 'yellow' ,'green');
	
	
	if(startflag==1){
		get_uart= Number(document.getElementById('data_text').innerText);
		
		display_text="<svg xmlns='http://www.w3.org/2000/svg' version='1.1' height='" + screen_h + "' width='" + screen_w + "' viewBox='-80 -10 1000 750' class='SvgFrame'>"
		display_text = display_text + "<rect x='-80' y='-10' width='1000' height='50' rx='10' ry='10' fill='blue' />"
		display_text = display_text + "<text x='0' y='30' font-size='40' stroke='cyan'  fill='cyan' text-anchor='start' stroke-width='1'>なぞりセンサ　DEMO</text>"
		display_text = display_text + "<text x='0' y='100' font-size='20' stroke='red'  fill='red' text-anchor='start' stroke-width='1'>ゆっくりなぞってください</text>"
		
		
		if(gesflag==1){
    		gescyc=0;
    		console.log("gesflag==1");
    		
		    if(sosimode==0){	
		      display_text = display_text + pos0_line();
		      display_text = display_text + pos2_line();
		      display_text = display_text + pos4_line();
		      display_text = display_text + pos6_line();
		      display_text = display_text + pos8_line();
		    }else if(sosimode==1){
		      display_text = display_text + pos0_line();
		      display_text = display_text + pos1_line();
		      display_text = display_text + pos2_line();
		      display_text = display_text + pos3_line();
		      display_text = display_text + pos4_line();
		      display_text = display_text + pos5_line();
		      display_text = display_text + pos6_line();
		      display_text = display_text + pos7_line();
		      display_text = display_text + pos8_line();
		    }
		    
		    gesflag=2;
		    display_text += "</svg>"
		    document.getElementById("svg").innerHTML =  display_text;
    	}
    	
    	
    	
		if(p_pos != get_uart){
			gesflag=0;
			gescyc=0;
			
			
		    if(sosimode==0){
		      if(get_uart==0){
		        display_text = display_text + pos0_line();
		        display_text = display_text + pos2_line();
		        display_text = display_text + pos4_line();
		        display_text = display_text + pos6_line();
		        display_text = display_text + pos8_line();  
		      }else if(get_uart==1){
		        display_text = display_text + pos2_line();
		        display_text = display_text + pos4_line();
		        display_text = display_text + pos6_line();
				display_text = display_text +  pos8_line();
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX-17*baisu)+","+(lineY-85*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY-85*baisu)+" "+(lineX+0*baisu)+","+(lineY-130*baisu)+" "+(lineX+32*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
		      }else if(get_uart==2){
		        display_text = display_text + pos0_line();
		        display_text = display_text + pos4_line();
		        display_text = display_text + pos6_line();
		        display_text = display_text + pos8_line();
		        display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+50*baisu)+","+(lineY+17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX+85*baisu)+","+(lineY-32*baisu)+" "+(lineX+130*baisu)+","+(lineY-0*baisu)+" "+(lineX+85*baisu)+","+(lineY+32*baisu)+"' stroke='orange' fill='orange' />"
		      }else if(get_uart==3){
		        display_text = display_text + pos0_line();
		        display_text = display_text + pos2_line();
		        display_text = display_text + pos6_line();
		        display_text = display_text + pos8_line();
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX-17*baisu)+","+(lineY+85*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY+85*baisu)+" "+(lineX+0*baisu)+","+(lineY+130*baisu)+" "+(lineX+32*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"  
		      }else if(get_uart==4){
		        display_text = display_text + pos0_line();
		        display_text = display_text + pos2_line();
		        display_text = display_text + pos4_line();
		        display_text = display_text + pos8_line();
		        display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-50*baisu)+","+(lineY+17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-85*baisu)+","+(lineY-32*baisu)+" "+(lineX-130*baisu)+","+(lineY+0*baisu)+" "+(lineX-85*baisu)+","+(lineY+32*baisu)+"' stroke='orange' fill='orange' />"     
		      }else if(get_uart==5){
		        display_text = display_text + pos0_line();
		        display_text = display_text + pos2_line();
		        display_text = display_text + pos4_line();
		        display_text = display_text + pos6_line();
				display_text = display_text + "<circle cx='"+lineX+"' cy='"+lineY+"' r='" +30*baisu+"' fill='orange' />"
		      }else if(get_uart==20){
		        display_text = display_text + pos8_line();
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX-17*baisu)+","+(lineY-85*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY-85*baisu)+" "+(lineX+0*baisu)+","+(lineY-130*baisu)+" "+(lineX+32*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
		        
		        display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+50*baisu)+","+(lineY+17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX+85*baisu)+","+(lineY-32*baisu)+" "+(lineX+130*baisu)+","+(lineY-0*baisu)+" "+(lineX+85*baisu)+","+(lineY+32*baisu)+"' stroke='orange' fill='orange' />"
		        
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX-17*baisu)+","+(lineY+85*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY+85*baisu)+" "+(lineX+0*baisu)+","+(lineY+130*baisu)+" "+(lineX+32*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
		        
		        display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-50*baisu)+","+(lineY+17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />";
		        display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
		        display_text = display_text + "<polygon points='"+(lineX-85*baisu)+","+(lineY-32*baisu)+" "+(lineX-130*baisu)+","+(lineY+0*baisu)+" "+(lineX-85*baisu)+","+(lineY+32*baisu)+"' stroke='orange' fill='orange' />"
		      
		      }else if(get_uart==21){
		     	display_text = display_text + "<circle cx='"+lineX+"' cy='"+lineY+"' r='" +29*baisu+"' fill='orangered' />"
		        display_text = display_text + pos0_line();
		        display_text = display_text + pos2_line();
		        display_text = display_text + pos4_line();
		        display_text = display_text + pos6_line();
		      }
		    }else if(sosimode==1){
				DisplayB();
		    }
		    
		    display_text += "</svg>"
	    	document.getElementById("svg").innerHTML =  display_text;
		}else{
			gescyc++;
		    if(gescyc>300) gescyc=400;
		    if(gescyc>300 && gesflag==0) gesflag=1;
		}
		
		p_pos=get_uart;
		pre_sosimode=sosimode;
		
	    
    }
}



function display0(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();  
}

function display1(){
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX-17*baisu)+","+(lineY-85*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY-85*baisu)+" "+(lineX+0*baisu)+","+(lineY-130*baisu)+" "+(lineX+32*baisu)+","+(lineY-85*baisu)+"' stroke='orange' fill='orange' />"
}

function display2(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+50*baisu)+","+(lineY+17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX+85*baisu)+","+(lineY-32*baisu)+" "+(lineX+130*baisu)+","+(lineY-0*baisu)+" "+(lineX+85*baisu)+","+(lineY+32*baisu)+"' stroke='orange' fill='orange' />"
}

function display3(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX-17*baisu)+","+(lineY+85*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY+85*baisu)+" "+(lineX+0*baisu)+","+(lineY+130*baisu)+" "+(lineX+32*baisu)+","+(lineY+85*baisu)+"' stroke='orange' fill='orange' />"   
}

function display4(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-50*baisu)+","+(lineY+17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-85*baisu)+","+(lineY-32*baisu)+" "+(lineX-130*baisu)+","+(lineY+0*baisu)+" "+(lineX-85*baisu)+","+(lineY+32*baisu)+"' stroke='orange' fill='orange' />"
}

function display5(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + "<circle cx='"+lineX+"' cy='"+lineY+"' r='" +30*baisu+"' fill='orange' />";
}

function display6(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX+23*baisu)+","+(lineY-47*baisu)+" "+(lineX+47*baisu)+","+(lineY-23*baisu)+" "+(lineX+48*baisu)+","+(lineY-72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX+72*baisu)+","+(lineY-48*baisu)+" "+(lineX+47*baisu)+","+(lineY-23*baisu)+" "+(lineX+48*baisu)+","+(lineY-72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX+37*baisu)+","+(lineY-83*baisu)+" "+(lineX+82*baisu)+","+(lineY-37*baisu)+" "+(lineX+92*baisu)+","+(lineY-92*baisu)+"' stroke='orange' fill='orange' />"
}

function display7(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX+23*baisu)+","+(lineY+47*baisu)+" "+(lineX+47*baisu)+","+(lineY+23*baisu)+" "+(lineX+48*baisu)+","+(lineY+72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX+72*baisu)+","+(lineY+48*baisu)+" "+(lineX+47*baisu)+","+(lineY+23*baisu)+" "+(lineX+48*baisu)+","+(lineY+72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX+37*baisu)+","+(lineY+83*baisu)+" "+(lineX+82*baisu)+","+(lineY+37*baisu)+" "+(lineX+92*baisu)+","+(lineY+92*baisu)+"' stroke='orange' fill='orange' />"
}

function display8(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX-23*baisu)+","+(lineY+47*baisu)+" "+(lineX-47*baisu)+","+(lineY+23*baisu)+" "+(lineX-48*baisu)+","+(lineY+72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-72*baisu)+","+(lineY+48*baisu)+" "+(lineX-47*baisu)+","+(lineY+23*baisu)+" "+(lineX-48*baisu)+","+(lineY+72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-37*baisu)+","+(lineY+83*baisu)+" "+(lineX-82*baisu)+","+(lineY+37*baisu)+" "+(lineX-92*baisu)+","+(lineY+92*baisu)+"' stroke='orange' fill='orange' />"
}

function display9(){
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX-23*baisu)+","+(lineY-47*baisu)+" "+(lineX-47*baisu)+","+(lineY-23*baisu)+" "+(lineX-48*baisu)+","+(lineY-72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-72*baisu)+","+(lineY-48*baisu)+" "+(lineX-47*baisu)+","+(lineY-23*baisu)+" "+(lineX-48*baisu)+","+(lineY-72*baisu)+"' stroke='orange' fill='orange' />"
	display_text = display_text + "<polygon points='"+(lineX-37*baisu)+","+(lineY-83*baisu)+" "+(lineX-82*baisu)+","+(lineY-37*baisu)+" "+(lineX-92*baisu)+","+(lineY-92*baisu)+"' stroke='orange' fill='orange' />"
}

function display20(){
	display_text = display_text + pos8_line();
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-50*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY-50*baisu)+" "+(lineX-17*baisu)+","+(lineY-85*baisu)+" "+(lineX+17*baisu)+","+(lineY-85*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY-85*baisu)+" "+(lineX-0*baisu)+","+(lineY-130*baisu)+" "+(lineX+32*baisu)+","+(lineY-85*baisu)+"' stroke='orangered' fill='orangered' />"          

	display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+50*baisu)+","+(lineY+17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX+50*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY-17*baisu)+" "+(lineX+85*baisu)+","+(lineY+17*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX+85*baisu)+","+(lineY-32*baisu)+" "+(lineX+130*baisu)+","+(lineY-0*baisu)+" "+(lineX+85*baisu)+","+(lineY+32*baisu)+"' stroke='orangered' fill='orangered' />"         
	
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+50*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-17*baisu)+","+(lineY+50*baisu)+" "+(lineX-17*baisu)+","+(lineY+85*baisu)+" "+(lineX+17*baisu)+","+(lineY+85*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-32*baisu)+","+(lineY+85*baisu)+" "+(lineX+0*baisu)+","+(lineY+130*baisu)+" "+(lineX+32*baisu)+","+(lineY+85*baisu)+"' stroke='orangered' fill='orangered' />"
	
	display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-50*baisu)+","+(lineY+17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-50*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY-17*baisu)+" "+(lineX-85*baisu)+","+(lineY+17*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-85*baisu)+","+(lineY-32*baisu)+" "+(lineX-130*baisu)+","+(lineY-0*baisu)+" "+(lineX-85*baisu)+","+(lineY+32*baisu)+"' stroke='orangered' fill='orangered' />"
	
	display_text = display_text + "<polygon points='"+(lineX+23*baisu)+","+(lineY-47*baisu)+" "+(lineX+47*baisu)+","+(lineY-23*baisu)+" "+(lineX+48*baisu)+","+(lineY-72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX+72*baisu)+","+(lineY-48*baisu)+" "+(lineX+47*baisu)+","+(lineY-23*baisu)+" "+(lineX+48*baisu)+","+(lineY-72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX+37*baisu)+","+(lineY-83*baisu)+" "+(lineX+82*baisu)+","+(lineY-37*baisu)+" "+(lineX+92*baisu)+","+(lineY-92*baisu)+"' stroke='orangered' fill='orangered' />"  
	
	display_text = display_text + "<polygon points='"+(lineX+23*baisu)+","+(lineY+47*baisu)+" "+(lineX+47*baisu)+","+(lineY+23*baisu)+" "+(lineX+48*baisu)+","+(lineY+72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX+72*baisu)+","+(lineY+48*baisu)+" "+(lineX+47*baisu)+","+(lineY+23*baisu)+" "+(lineX+48*baisu)+","+(lineY+72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX+37*baisu)+","+(lineY+83*baisu)+" "+(lineX+82*baisu)+","+(lineY+37*baisu)+" "+(lineX+92*baisu)+","+(lineY+92*baisu)+"' stroke='orangered' fill='orangered' />"

	display_text = display_text + "<polygon points='"+(lineX-23*baisu)+","+(lineY+47*baisu)+" "+(lineX-47*baisu)+","+(lineY+23*baisu)+" "+(lineX-48*baisu)+","+(lineY+72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-72*baisu)+","+(lineY+48*baisu)+" "+(lineX-47*baisu)+","+(lineY+23*baisu)+" "+(lineX-48*baisu)+","+(lineY+72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-37*baisu)+","+(lineY+83*baisu)+" "+(lineX-82*baisu)+","+(lineY+37*baisu)+" "+(lineX-92*baisu)+","+(lineY+92*baisu)+"' stroke='orangered' fill='orangered' />"
	
	display_text = display_text + "<polygon points='"+(lineX-23*baisu)+","+(lineY-47*baisu)+" "+(lineX-47*baisu)+","+(lineY-23*baisu)+" "+(lineX-48*baisu)+","+(lineY-72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-72*baisu)+","+(lineY-48*baisu)+" "+(lineX-47*baisu)+","+(lineY-23*baisu)+" "+(lineX-48*baisu)+","+(lineY-72*baisu)+"' stroke='orangered' fill='orangered' />"
	display_text = display_text + "<polygon points='"+(lineX-37*baisu)+","+(lineY-83*baisu)+" "+(lineX-82*baisu)+","+(lineY-37*baisu)+" "+(lineX-92*baisu)+","+(lineY-92*baisu)+"' stroke='orangered' fill='orangered' />"
}

function display21(){
	display_text = display_text + "<circle cx='"+lineX+"' cy='"+lineY+"' r='" +29*baisu+"' fill='orangered' />";  
	display_text = display_text + pos0_line();
	display_text = display_text + pos1_line();
	display_text = display_text + pos2_line();
	display_text = display_text + pos3_line();
	display_text = display_text + pos4_line();
	display_text = display_text + pos5_line();
	display_text = display_text + pos6_line();
	display_text = display_text + pos7_line();
}








function DisplayB(){
	if(get_uart==0){
		display0();
	}else if(get_uart==1){
		display1();
	}else if(get_uart==2){
		display2();
	}else if(get_uart==3){
		display3();
	}else if(get_uart==4){
		display4();
	}else if(get_uart==6){
		display6();
	}else if(get_uart==7){
		display7();
	}else if(get_uart==8){
		display8();
	}else if(get_uart==9){
        display9();
	}else if(get_uart==5){
		display5();
	}else if(get_uart==20){
		display20();
	
	}else if(get_uart==21){
		display21()
	}
}




function pos0_line(){
	return	"<polyline points='"+
	(lineX-17*baisu)+","+(lineY-50*baisu)+" "+
	(lineX+17*baisu)+","+(lineY-50*baisu)+" "+
	(lineX+17*baisu)+","+(lineY-85*baisu)+" "+
	(lineX+32*baisu)+","+(lineY-85*baisu)+" "+
	(lineX+0*baisu)+","+(lineY-130*baisu)+" "+
	(lineX-32*baisu)+","+(lineY-85*baisu)+" "+
	(lineX-17*baisu)+","+(lineY-85*baisu)+" "+
	(lineX-17*baisu)+","+(lineY-50*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos1_line(){
	return	"<polyline points='"+
	(lineX+23*baisu)+","+(lineY-47*baisu)+" "+
	(lineX+47*baisu)+","+(lineY-23*baisu)+" "+
	(lineX+72*baisu)+","+(lineY-48*baisu)+" "+
	(lineX+82*baisu)+","+(lineY-37*baisu)+" "+
	(lineX+92*baisu)+","+(lineY-92*baisu)+" "+
	(lineX+37*baisu)+","+(lineY-83*baisu)+" "+
	(lineX+48*baisu)+","+(lineY-72*baisu)+" "+
	(lineX+23*baisu)+","+(lineY-47*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos2_line(){
	return	"<polyline points='"+
	(lineX+50*baisu)+","+(lineY-17*baisu)+" "+
	(lineX+50*baisu)+","+(lineY+17*baisu)+" "+
	(lineX+85*baisu)+","+(lineY+17*baisu)+" "+
	(lineX+85*baisu)+","+(lineY+32*baisu)+" "+
	(lineX+130*baisu)+","+(lineY+0*baisu)+" "+
	(lineX+85*baisu)+","+(lineY-32*baisu)+" "+
	(lineX+85*baisu)+","+(lineY-17*baisu)+" "+
	(lineX+50*baisu)+","+(lineY-17*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos3_line(){
	return	"<polyline points='"+
	(lineX+47*baisu)+","+(lineY+23*baisu)+" "+
	(lineX+23*baisu)+","+(lineY+47*baisu)+" "+
	(lineX+48*baisu)+","+(lineY+72*baisu)+" "+
	(lineX+34*baisu)+","+(lineY+83*baisu)+" "+
	(lineX+92*baisu)+","+(lineY+92*baisu)+" "+
	(lineX+83*baisu)+","+(lineY+37*baisu)+" "+
	(lineX+72*baisu)+","+(lineY+48*baisu)+" "+
	(lineX+47*baisu)+","+(lineY+23*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos4_line(){
	return	"<polyline points='"+
	(lineX-17*baisu)+","+(lineY+50*baisu)+" "+
	(lineX+17*baisu)+","+(lineY+50*baisu)+" "+
	(lineX+17*baisu)+","+(lineY+85*baisu)+" "+
	(lineX+32*baisu)+","+(lineY+85*baisu)+" "+
	(lineX+0*baisu)+","+(lineY+130*baisu)+" "+
	(lineX-32*baisu)+","+(lineY+85*baisu)+" "+
	(lineX-17*baisu)+","+(lineY+85*baisu)+" "+
	(lineX-17*baisu)+","+(lineY+50*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos5_line(){
	return	"<polyline points='"+
	(lineX-47*baisu)+","+(lineY+23*baisu)+" "+
	(lineX-23*baisu)+","+(lineY+47*baisu)+" "+
	(lineX-48*baisu)+","+(lineY+72*baisu)+" "+
	(lineX-34*baisu)+","+(lineY+83*baisu)+" "+
	(lineX-92*baisu)+","+(lineY+92*baisu)+" "+
	(lineX-83*baisu)+","+(lineY+37*baisu)+" "+
	(lineX-72*baisu)+","+(lineY+48*baisu)+" "+
	(lineX-47*baisu)+","+(lineY+23*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos6_line(){
	return	"<polyline points='"+
	(lineX-50*baisu)+","+(lineY-17*baisu)+" "+
	(lineX-50*baisu)+","+(lineY+17*baisu)+" "+
	(lineX-85*baisu)+","+(lineY+17*baisu)+" "+
	(lineX-85*baisu)+","+(lineY+32*baisu)+" "+
	(lineX-130*baisu)+","+(lineY+0*baisu)+" "+
	(lineX-85*baisu)+","+(lineY-32*baisu)+" "+
	(lineX-85*baisu)+","+(lineY-17*baisu)+" "+
	(lineX-50*baisu)+","+(lineY-17*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos7_line(){
	return	"<polyline points='"+
	(lineX-23*baisu)+","+(lineY-47*baisu)+" "+
	(lineX-47*baisu)+","+(lineY-23*baisu)+" "+
	(lineX-72*baisu)+","+(lineY-48*baisu)+" "+
	(lineX-82*baisu)+","+(lineY-37*baisu)+" "+
	(lineX-92*baisu)+","+(lineY-92*baisu)+" "+
	(lineX-37*baisu)+","+(lineY-83*baisu)+" "+
	(lineX-48*baisu)+","+(lineY-72*baisu)+" "+
	(lineX-23*baisu)+","+(lineY-47*baisu)+
	"' stroke='fuchsia' fill='none' />";
}

function pos8_line(){
	return "<circle cx='"+lineX+"' cy='"+lineY+"' r='" +30*baisu+"' stroke='fuchsia' fill='none' />";
}

</script>
</body>
</html>
