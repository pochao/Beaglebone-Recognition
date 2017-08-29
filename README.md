# Beaglebone-Recognition

```
var request = require('request');
var fs = require('fs');
var googleTTS = require('google-tts-api');
var Button = require('./grove_button');
var button = new Button('P9_22');
var Sound = require('./node_arecord')
var sound = new Sound('my.wave');
var spawn = require('child_process').spawn;
var LCD = require('jsupm_i2clcd');
var myLcd = new LCD.SSD1327 (1, 0x3C);
myLcd.setGrayLevel(12);
var record_state = 0;

displayLed('Ready');
button.on('click',function(){
    if (record_state){
        sound.stop();
    } else {
        displayLed('Listening');
        record_state = 1;
        sound = new Sound('my.wave');
        
        sound.record();
        
        sound.on('complete',function(){
            console.log('錄製完成');
            displayLed('Record ok');
            record_state = 0;
            
            witASR('my.wave').then(function(result){
                var txt = result._text;
                displayLed('Recognition');
                console.log(txt);
                return googleTTS(txt, 'zh');
            }).then(function(result){
                console.log(result);
                
                return downloadFile(result,'my.mp3');
            }).then(function(){
                console.log('下載完成');
                displayLed('Download ok');
                
                spawn('mpg123', ['my.mp3']);
                displayLed('Ready');
            }).catch(function(){
                console.log('失敗');
                displayLed('Fail');
            })
            
        })
      
    }
})

function witASR(fileName){
	return new Promise(function(resolve, reject){
	    fs.readFile(fileName, function (err, file) {
	    	if (err) {
	    	  reject(err);
	    	}
	  
	    	var options = { method: 'POST',
	    	  url: 'https://api.wit.ai/speech',
	    	  headers: 
	    	   { 'postman-token': '1df96711-ff4c-7827-a62a-b98938ac5a2c',
	    	     'cache-control': 'no-cache',
	    	     'content-type': 'audio/wav',
	    	     authorization: 'Bearer KRxxxxxxxxxxxxxxxxxxxxxxxxDPSI' },
	    	   body: file };
	    
	    	request(options, function (err, response, body) {
	    	  	if (err) {
	    	  	  reject(err);
	    	  	}
	    	  	
	    	  	var jbody = JSON.parse(body);
	    	  	resolve(jbody);
	    	})
	    })      
	})
}


function downloadFile(filePath,saveFile){
  return new Promise(function(resolve, reject){
    request.get(filePath).pipe(fs.createWriteStream(saveFile).on('finish',function(err){
      if (err) { reject() }
        resolve();
    }))
  })
}

function displayLed(msg){
    myLcd.clear();
    myLcd.setGrayLevel(12);
    myLcd.setCursor(1, 0);
    myLcd.write(msg);
}
```
