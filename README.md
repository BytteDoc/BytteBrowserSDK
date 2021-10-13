# Bytte Browser SDK

![N|Solid](https://www.bytte.com.co/pagweb/wp-content/uploads/2019/05/by-casb-logo.png)

**Bytte Browser SDK** is an SDK that provides the functionality to perform different captures (documents, face, voice) from a web browser 

## Features

 - [Colombia Document Front Capture](#CaptureCOBackDocument).
 - [Colombia Document Back Capture](#CaptureCOFrontDocument).
 - [Colombia Digital Document Front Capture](#CaptureCODigitalFrontDocument).
 - [Colombia Digital Document Back Capture](#CaptureCODigitalBackDocument)
 - [Voice Capture](#VoiceCapture)
 - [Capture Result object](#CaptureDocumentoResult)
 - [Finger Capture](#CaptureFinger)
 - [Face Capture](#CaptureFace)

**Bytte Browser SDK**  is a commercial product, to download and use it contact info@bytte.com.co for instructions on how to obtain the required resources and access
 
## Npm Permissions
in your [.npmrc](https://go.microsoft.com/fwlink/?linkid=724423) file

```Shell
; begin auth token  
//byttetfs.pkgs.visualstudio.com/CASB/_packaging/bytte-browser-sdk/npm/registry/:username=BytteTFS  
//byttetfs.pkgs.visualstudio.com/CASB/_packaging/bytte-browser-sdk/npm/registry/:_password=[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]  
//byttetfs.pkgs.visualstudio.com/CASB/_packaging/bytte-browser-sdk/npm/registry/:email=npm requires email to be set but doesn't use the value  
//byttetfs.pkgs.visualstudio.com/CASB/_packaging/bytte-browser-sdk/npm/:username=BytteTFS  
//byttetfs.pkgs.visualstudio.com/CASB/_packaging/bytte-browser-sdk/npm/:_password=[BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]  
//byttetfs.pkgs.visualstudio.com/CASB/_packaging/bytte-browser-sdk/npm/:email=npm requires email to be set but doesn't use the value  
; end auth token
```
replace [BASE64_ENCODED_PERSONAL_ACCESS_TOKEN]  with a valid token !

## Installation

in your project already created, Install the dependencies and devDependencies and start the server.

```sh
cd my-project
npm install @bytte/byttebrowsersdk@2.0.5
npm install
```
Start the server
```bash
npm run start (or) ng serve
```

## Use Captures
## <a name="CaptureCOBackDocument"></a>CaptureCOBackDocument
Colombia Back Document with PDF417 code

Example Layout HTML
```html
<p>capture document example </p>

<div id="screen-initial">
    <h1 id="msg">Loading...</h1>
    <progress id="load-progress" value="0" max="100"></progress>
</div> 

<div id="screen-start" class="hidden">    
    <button id="btnStartCapture" (click)="startCapture()" value="Start scan..." class="btn btn-primary">Capturar Documento</button>
    <button id="btnClean" (click)="cleanMem()" value="CleanMem" class="btn btn-primary">CleanMem</button>
</div>

<div id="screen-scanning" class="hidden">
    <video id="camera-feed" playsinline></video>
    <!-- Recognition events will be drawn here. -->
    <canvas id="camera-feedback"></canvas>
    <!-- Recognition events will be drawn here. -->
    <p id="camera-guides">set document in front of back camera</p>
</div>

 <!-- Results. -->
<div style="display:block;"> 
    <div id="dvResult">
        <img id="imgResult" />        
    </div>
    <br />
    <label id="lblBarCode">--</label><br />
    <label id="lblMensaje">--</label><br />    
</div>

```

## Javascript/TypeScript Code  

```TypeScript
import  * as BytteSDK  from  "@bytte/byttebrowsersdk";
import  {  BaseReturn,  CaptureDocumentoResult  }  from  '@bytte/byttebrowsersdk';
...
...  

//1. Declare CaptureCOBackDocument object
private oSDKCapturaReversoDoc:  BytteSDK.CaptureCOBackDocument;  

//2. Set License Key (From Bytte)
private licenseKey =  "sR...";  

//3. Get HTML reference objects
public initialMessageEl:  HTMLHeadingElement;
public progressEl:  HTMLProgressElement;
public cameraFeed:  HTMLVideoElement;
public cameraFeedback:  HTMLCanvasElement;
public drawContext:  CanvasRenderingContext2D;
public scanFeedback:  HTMLParagraphElement;
private oImgResult:  HTMLImageElement;
...
...
this.initialMessageEl = document.getElementById("msg") as HTMLHeadingElement;
this.progressEl = document.getElementById("load-progress") as HTMLProgressElement;
this.cameraFeed = document.getElementById("camera-feed") as HTMLVideoElement;
this.cameraFeedback = document.getElementById("camera-feedback") as HTMLCanvasElement;
this.drawContext =  this.cameraFeedback.getContext("2d") as CanvasRenderingContext2D;
this.scanFeedback = document.getElementById("camera-guides") as HTMLParagraphElement;
this.oImgResult = document.getElementById("imgResult") as HTMLImageElement;
 
//4. Instance SDK
this.oSDKCapturaReversoDoc =  new  BytteSDK.CaptureCOBackDocument(licenseKey,
this.cameraFeed,
this.cameraFeedback,
this.drawContext,
this.scanFeedback,
false,  //Return base64 Image for test only!!
this.sCamInfo);  
```
> Return image Base64 string **IS NOT RECOMENDED**, use only for test
> SDK return a blob object, this contains the image
```TypeScript

//5. Set Time Out (example 30 segs)
this.oSDKCapturaReversoDoc.setTimeOut(30  *  1000);  

//6. Initialize SDK component WASM
this.oSDKCapturaReversoDoc.initSDK().then(oStatus =>
{
    //Succes Init
    this.status = oStatus.messaje;
    console.log(this.status);  
}).catch(oErr =>
{
    //Error Init SDK
    let oError = oErr as BaseReturn;
    //Fail    
});
  
//7. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oSDKCapturaReversoDoc.addCallBackStreamReady((streamReady: boolean, displayInfo?: string)  =>
{
    console.log("Stream Ready", streamReady);
    //Ready to Start!!
});  

```
> Call Capture function! 
> This procedure activate the back device Camera
> and Cavas object

```TypeScript

//8. Start Capture!!
...
this.oSDKCapturaReversoDoc.startCapture().then(oCapturaDocumentoResult => 
{ 
    //Function handle OK...
    this.ProcessCaptureOK(oCapturaDocumentoResult); 
})
.catch(oCapturaDocumentoResult => 
{ 
    //Function Handle Error...
    this.ProcesaCaptureError(oCapturaDocumentoResult); 
})

//Handle success
async ProcessCaptureOK(oDocumentoResult: CaptureDocumentoResult) 
{
    //if in the constructor, flag return imageBase64 is true, 
    //**return imageBase64 IS NOT RECOMENDED**
    documentImageBase64 field contains a image
    this.oImgResult.src = "data:image/png;base64," + oDocumentoResult.documentImageBase64;

    //BarCode Data in String
    document.getElementById("lblBarCode").innerHTML = "" + oDocumentoResult.barCodeBase64;

    //Process Message 
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;

    //Send instruction to SDK to End Capture
    //End stream and free memory resources
    this.oSDKCapturaReversoDoc.endCapture().then(() => {
        console.log("Fin Captura");
        })
}

//Handle Error
async ProcesaCaptureError(oDocumentoResult: CaptureDocumentoResult) {
    //Show error in div..
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;
} 

/* if you need a clean image of base64 memory
   when the mobile device is low on memory 
*/
cleanMem() {   
    this.oImgResult.src = "";
}

```
## ----------------------
## <a name="CaptureCOFrontDocument"></a>CaptureCOFrontDocument
Colombia Front Document

Example Layout HTML
```html
<p>capture document front component example!</p>
<div id="screen-initial">
    <h1 id="msg">Loading...</h1>
    <progress id="load-progress" value="0" max="100"></progress>
</div> 

<div id="screen-start" class="hidden">    
    <button id="btnStartCapture" (click)="startCapture()" value="Start Capture..." class="btn btn-primary">Capture</button>
    <button id="btnClean" (click)="cleanMem()" value="CleanMem" class="btn btn-primary">CleanMem</button>
</div>

<!-- Capture Controller -->
<div id="screen-scanning" class="hidden">
    <video id="camera-feed" playsinline></video>
    <!-- Recognition events will be drawn here. -->
    <canvas id="camera-feedback"></canvas>
    <p id="camera-guides">Set document in front of back camera</p>
</div>
<!-- Capture Controller -->

<!-- Result panel -->
<div style="display:block;" *ngIf="documentResult"> 
    <div id="dvResult">
        <img id="imgResult" [src]="imSrc" />        
    </div>
    <br />
    <label id="lblPrimerNombre">{{documentResult.firstName}}</label><br />
    <label id="lblSegundoNombre">{{documentResult.lastName}}</label><br />
    <label id="lblNumeroDocumento">{{documentResult.documentNumber}}</label><br />      
</div>
<label id="lblMensaje" *ngIf="mensaje!=''">{{mensaje}}</label><br /> 
```

## Javascript/TypeScript Code  

```TypeScript
import * as BytteSDK from "@bytte/byttebrowsersdk";
import { BaseReturn, CaptureDocumentoResult } from '@bytte/byttebrowsersdk';
...
...  
//1. Declare CaptureCOFrontDocument object
private oSDKCapturaFrenteDoc: BytteSDK.CaptureCOFrontDocument;

//2. Set License Key (From Bytte)
private licenseKey =  "sR...";  

//3. Get HTML reference objects
public initialMessageEl:  HTMLHeadingElement;
public progressEl:  HTMLProgressElement;
public cameraFeed:  HTMLVideoElement;
public cameraFeedback:  HTMLCanvasElement;
public drawContext:  CanvasRenderingContext2D;
public scanFeedback:  HTMLParagraphElement;
private oImgResult:  HTMLImageElement;
...
...
this.initialMessageEl = document.getElementById("msg") as HTMLHeadingElement;
this.progressEl = document.getElementById("load-progress") as HTMLProgressElement;
this.cameraFeed = document.getElementById("camera-feed") as HTMLVideoElement;
this.cameraFeedback = document.getElementById("camera-feedback") as HTMLCanvasElement;
this.drawContext =  this.cameraFeedback.getContext("2d") as CanvasRenderingContext2D;
this.scanFeedback = document.getElementById("camera-guides") as HTMLParagraphElement;
this.oImgResult = document.getElementById("imgResult") as HTMLImageElement;
 
//4. Instance SDK
this.oSDKCapturaFrenteDoc =  new  BytteSDK.CaptureCOFrontDocument(licenseKey,
this.cameraFeed,
this.cameraFeedback,
this.drawContext,
this.scanFeedback,
false,  //Return base64 Image for test only!!
this.sCamInfo);  
```
> Return image Base64 string **IS NOT RECOMENDED**, use only for test
> SDK return a blob object, this contains the image
```TypeScript

//5. Set Time Out (example 30 segs)
this.oSDKCapturaFrenteDoc.setTimeOut(30  *  1000);  

//6. Initialize SDK component WASM
this.oSDKCapturaFrenteDoc.initSDK().then(oStatus =>
{
    //Succes Init
    this.status = oStatus.messaje;
    console.log(this.status);  
}).catch(oErr =>
{
    //Error Init SDK
    let oError = oErr as BaseReturn;
    //Fail    
});
  
//7. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oSDKCapturaFrenteDoc.addCallBackStreamReady((streamReady: boolean, displayInfo?: string)  =>
{
    console.log("Stream Ready", streamReady);
    //Ready to Start!!
});  

```
> Call Capture function! 
> This procedure activate the back device Camera
> and Cavas object

```TypeScript

//8. Start Capture!!
...
this.oSDKCapturaFrenteDoc.startCapture().then(oCapturaDocumentoResult => 
{ 
    //Function handle OK...
    this.ProcessCaptureOK(oCapturaDocumentoResult); 
})
.catch(oCapturaDocumentoResult => 
{ 
    //Function Handle Error...
    this.ProcesaCaptureError(oCapturaDocumentoResult); 
})

//Handle success
async ProcessCaptureOK(oDocumentoResult: CaptureDocumentoResult) 
{
    //if in the constructor, flag return imageBase64 is true, 
    //**return imageBase64 IS NOT RECOMENDED**
    documentImageBase64 field contains a image
    this.oImgResult.src = "data:image/png;base64," + oDocumentoResult.documentImageBase64;
  
    //Process Message 
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;

    //This fields return information!!
    // --> oDocumentoResult.firstName;
    // --> oDocumentoResult.lastName;
    // --> oDocumentoResult.documentNumber;

    //Send instruction to SDK to End Capture
    //End stream and free memory resources
    this.oSDKCapturaFrenteDoc.endCapture().then(() => {
        console.log("Fin Captura");
        })
}

//Handle Error
async ProcesaCaptureError(oDocumentoResult: CaptureDocumentoResult) {
    //Show error in div..
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;
} 

/* if you need a clean image of base64 memory
   when the mobile device is low on memory 
*/
cleanMem() {
    this.oImgResult.src = "";
}

```

## <a name="CaptureCODigitalBackDocument"></a>CaptureCODigitalBackDocument (New Digital Document)
Colombia Back Document with MRZ code

Example Layout HTML
```html
<p>capture digital back Document test</p>

<div id="screen-initial">
    <h1 id="msg">Loading...</h1>
    <progress id="load-progress" value="0" max="100"></progress>
</div> 

<div id="screen-start" class="hidden">    
    <button id="btnStartCapture" (click)="startCapture()" value="Start scan..." class="btn btn-primary">Capture Document</button>
</div>

<!-- Capture Control. -->
<div id="screen-scanning" class="hidden">
    <video id="camera-feed" playsinline></video>
    <!-- Recognition events will be drawn here. -->
    <canvas id="camera-feedback"></canvas>
    <p id="camera-guides">Set the document in front of back camera</p>
</div>

<!-- Information panel. -->
<div style="display:block;"> 
    <div id="dvResult">
        <img id="imgResult" />        
    </div>
    <br />
    <label id="lblBarCode">--</label><br />
    <label id="lblMensaje">--</label><br />    
</div>
```

## Javascript/TypeScript Code  

```TypeScript
import * as BytteSDK from "@bytte/byttebrowsersdk";
import { BaseReturn, CaptureDocumentoResult } from '@bytte/byttebrowsersdk';
...
...  

//1. Declare CaptureCODigitalBackDocument object
private oSDKCapturaReversoDoc:  BytteSDK.CaptureCODigitalBackDocument;  

//2. Set License Key (From Bytte)
private licenseKey =  "sR...";  

//3. Get HTML reference objects
public initialMessageEl:  HTMLHeadingElement;
public progressEl:  HTMLProgressElement;
public cameraFeed:  HTMLVideoElement;
public cameraFeedback:  HTMLCanvasElement;
public drawContext:  CanvasRenderingContext2D;
public scanFeedback:  HTMLParagraphElement;
private oImgResult:  HTMLImageElement;
...
...
this.initialMessageEl = document.getElementById("msg") as HTMLHeadingElement;
this.progressEl = document.getElementById("load-progress") as HTMLProgressElement;
this.cameraFeed = document.getElementById("camera-feed") as HTMLVideoElement;
this.cameraFeedback = document.getElementById("camera-feedback") as HTMLCanvasElement;
this.drawContext =  this.cameraFeedback.getContext("2d") as CanvasRenderingContext2D;
this.scanFeedback = document.getElementById("camera-guides") as HTMLParagraphElement;
this.oImgResult = document.getElementById("imgResult") as HTMLImageElement;
 
//4. Instance SDK
this.oSDKCapturaReversoDoc =  new  BytteSDK.CaptureCODigitalBackDocument(licenseKey,
this.cameraFeed,
this.cameraFeedback,
this.drawContext,
this.scanFeedback,
false,  //Return base64 Image for test only!!
this.sCamInfo);  
```
> Return image Base64 string **IS NOT RECOMENDED**, use only for test
> SDK return a blob object, this contains the image
```TypeScript

//5. Set Time Out (example 30 segs)
this.oSDKCapturaReversoDoc.setTimeOut(30  *  1000);  

//6. Initialize SDK component WASM
this.oSDKCapturaReversoDoc.initSDK().then(oStatus =>
{
    //Succes Init
    this.status = oStatus.messaje;
    console.log(this.status);  
}).catch(oErr =>
{
    //Error Init SDK
    let oError = oErr as BaseReturn;
    //Fail    
});
  
//7. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oSDKCapturaReversoDoc.addCallBackStreamReady((streamReady: boolean, displayInfo?: string)  =>
{
    console.log("Stream Ready", streamReady);
    //Ready to Start!!
});  

```
> Call Capture function! 
> This procedure activate the back device Camera
> and Cavas object

```TypeScript

//8. Start Capture!!
...
this.oSDKCapturaReversoDoc.startCapture().then(oCapturaDocumentoResult => 
{ 
    //Function handle OK...
    this.ProcessCaptureOK(oCapturaDocumentoResult); 
})
.catch(oCapturaDocumentoResult => 
{ 
    //Function Handle Error...
    this.ProcesaCaptureError(oCapturaDocumentoResult); 
})

//Handle success
async ProcessCaptureOK(oDocumentoResult: CaptureDocumentoResult) 
{
    //if in the constructor, flag return imageBase64 is true, 
    //**return imageBase64 IS NOT RECOMENDED**
    documentImageBase64 field contains a image
    this.oImgResult.src = "data:image/png;base64," + oDocumentoResult.documentImageBase64;

    //BarCode MRZ Information Data in String
    document.getElementById("lblBarCode").innerHTML = "" + oDocumentoResult.barCodeBase64;

    //Process Message 
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;

    //Send instruction to SDK to End Capture
    //End stream and free memory resources
    this.oSDKCapturaReversoDoc.endCapture().then(() => {
        console.log("Fin Captura");
        })
}

//Handle Error
async ProcesaCaptureError(oDocumentoResult: CaptureDocumentoResult) {
    //Show error in div..
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;
} 

/* if you need a clean image of base64 memory
   when the mobile device is low on memory 
*/
cleanMem() {   
    this.oImgResult.src = "";
}

```
## ----------------------
## <a name="CaptureCODigitalFrontDocument"></a>CaptureCODigitalFrontDocument
Colombia digital Front Document (New Digital Document)

Example Layout HTML
```html
<p>capture document front digital</p>
<div id="screen-initial">
    <h1 id="msg">Loading...</h1>
    <progress id="load-progress" value="0" max="100"></progress>
</div> 

<div id="screen-start" class="hidden">    
    <button id="btnStartCapture" (click)="startCapture()" value="Start Capture..." class="btn btn-primary">Capture</button>
</div>

<!-- Capture Control. -->
<div id="screen-scanning" class="hidden">
    <video id="camera-feed" playsinline></video>
    <!-- Recognition events will be drawn here. -->
    <canvas id="camera-feedback"></canvas>
    <p id="camera-guides">Set the document in front of back camera!</p>
</div>

<!-- Result Panel -->
<div style="display:block;" *ngIf="documentResult"> 
    <div id="dvResult">
        <img id="imgResult" [src]="imSrc" />        
    </div>
    <br />
    <label id="lblPrimerNombre">{{documentResult.firstName}}</label><br />
    <label id="lblSegundoNombre">{{documentResult.lastName}}</label><br />
    <label id="lblNumeroDocumento">{{documentResult.documentNumber}}</label><br />      
</div>

<label id="lblMensaje" *ngIf="mensaje!=''">{{mensaje}}</label><br /> 
```

## Javascript/TypeScript Code  

```TypeScript
import * as BytteSDK from "@bytte/byttebrowsersdk";
import { BaseReturn, CaptureDocumentoResult } from '@bytte/byttebrowsersdk';
...
...  
//1. Declare CaptureCOFrontDocument object
private oSDKCapturaFrenteDoc: BytteSDK.CaptureCODigitalFrontDocument;

//2. Set License Key (From Bytte)
private licenseKey =  "sR...";  

//3. Get HTML reference objects
public initialMessageEl:  HTMLHeadingElement;
public progressEl:  HTMLProgressElement;
public cameraFeed:  HTMLVideoElement;
public cameraFeedback:  HTMLCanvasElement;
public drawContext:  CanvasRenderingContext2D;
public scanFeedback:  HTMLParagraphElement;
private oImgResult:  HTMLImageElement;
...
...
this.initialMessageEl = document.getElementById("msg") as HTMLHeadingElement;
this.progressEl = document.getElementById("load-progress") as HTMLProgressElement;
this.cameraFeed = document.getElementById("camera-feed") as HTMLVideoElement;
this.cameraFeedback = document.getElementById("camera-feedback") as HTMLCanvasElement;
this.drawContext =  this.cameraFeedback.getContext("2d") as CanvasRenderingContext2D;
this.scanFeedback = document.getElementById("camera-guides") as HTMLParagraphElement;
this.oImgResult = document.getElementById("imgResult") as HTMLImageElement;
 
//4. Instance SDK
this.oSDKCapturaFrenteDoc =  new  BytteSDK.CaptureCODigitalFrontDocument(licenseKey,
this.cameraFeed,
this.cameraFeedback,
this.drawContext,
this.scanFeedback,
false,  //Return base64 Image for test only!!
this.sCamInfo);  
```
> Return image Base64 string **IS NOT RECOMENDED**, use only for test
> SDK return a blob object, this contains the image
```TypeScript

//5. Set Time Out (example 30 segs)
this.oSDKCapturaFrenteDoc.setTimeOut(30  *  1000);  

//6. Initialize SDK component WASM
this.oSDKCapturaFrenteDoc.initSDK().then(oStatus =>
{
    //Succes Init
    this.status = oStatus.messaje;
    console.log(this.status);  
}).catch(oErr =>
{
    //Error Init SDK
    let oError = oErr as BaseReturn;
    //Fail    
});
  
//7. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oSDKCapturaFrenteDoc.addCallBackStreamReady((streamReady: boolean, displayInfo?: string)  =>
{
    console.log("Stream Ready", streamReady);
    //Ready to Start!!
});  

```
> Call Capture function! 
> This procedure activate the back device Camera
> and Cavas object

```TypeScript

//8. Start Capture!!
...
this.oSDKCapturaFrenteDoc.startCapture().then(oCapturaDocumentoResult => 
{ 
    //Function handle OK...
    this.ProcessCaptureOK(oCapturaDocumentoResult); 
})
.catch(oCapturaDocumentoResult => 
{ 
    //Function Handle Error...
    this.ProcesaCaptureError(oCapturaDocumentoResult); 
})

//Handle success
async ProcessCaptureOK(oDocumentoResult: CaptureDocumentoResult) 
{
    //if in the constructor, flag return imageBase64 is true, 
    //**return imageBase64 IS NOT RECOMENDED**
    documentImageBase64 field contains a image
    this.oImgResult.src = "data:image/png;base64," + oDocumentoResult.documentImageBase64;
  
    //Process Message 
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;

    //This fields return information!!
    // --> oDocumentoResult.firstName;
    // --> oDocumentoResult.lastName;
    // --> oDocumentoResult.documentNumber;

    //Send instruction to SDK to End Capture
    //End stream and free memory resources
    this.oSDKCapturaFrenteDoc.endCapture().then(() => {
        console.log("Fin Captura");
        })
}

//Handle Error
async ProcesaCaptureError(oDocumentoResult: CaptureDocumentoResult) {
    //Show error in div..
    document.getElementById("lblMensaje").innerHTML = "" + oDocumentoResult.messaje;
} 

/* if you need a clean image of base64 memory
   when the mobile device is low on memory 
*/
cleanMem() {
    this.oImgResult.src = "";
}

```

## ----------------------
## <a name="VoiceCapture"></a>VoiceCapture
Voice Capture Component - Capture microphone audio, then return this capture in mp3 audio file
HTMLAudioElement Can be hidden if don't require play the previous capture

Example Layout HTML
```html
<p>Capture Audio MP3!</p>
<div id="divCaptura">
    <button id="btnStartCapture" (click)="startCapture()" value="Start Capture..." class="btn btn-primary">Capture Audio</button>
</div>
<div id="divProcesando" *ngIf="inProcess" >
    <h1 id="msg">Processing ... {{currentTime}}/{{maxTime}} </h1>    
</div> 
<br />
<div id="divReproductor">Html5 - Capture Audio player!!</div>
```
## Javascript/TypeScript Code  

```TypeScript
import * as BytteSDK from "@bytte/byttebrowsersdk";
import { CaptureAudioResult } from '@bytte/byttebrowsersdk/types/DataTypes/CaptureAudioResult';
...
...  
//1. Declare CaptureFace object
private oSDKCapturaVoice : BytteSDK.CaptureVoice;  

//2. Get HTML reference objects
private player           : HTMLAudioElement;
private divPlayer        : HTMLDivElement;
public maxTime           : number;
public currentTime       : number;  
public inProcess         : boolean; 
...
...
this.divPlayer = document.getElementById("divReproductor") as HTMLDivElement; 
 
//3. Instance SDK
this.oSDKCapturaVoice = new BytteSDK.CaptureVoice();
 
//4. Set Processing Timer info callback
this.oSDKCapturaVoice.addCallBackVozTimer((maxTime: number, currentTime: number) => this.addCallBackVozTimer (maxTime, currentTime));

/* Callback Process Timer */
private addCallBackVozTimer(maxTime: number, currentTime: number){
    this.maxTime     = maxTime;
    this.currentTime = currentTime;
} 
 
```
> Call Capture function! 
> This procedure activate the microphone

```TypeScript
//5. Start Capture!!
//Reset Flags and values in 0
this.inProcess   = true;
this.maxTime     = 0;
this.currentTime = 0;    

//If Divplayer already exists, remove it 
if (this.divPlayer.childNodes.length > 0){
    this.divPlayer.childNodes.forEach(x => { this.divPlayer.removeChild(x); })
}

/* Start Capture voice Process */
this.oSDKCapturaVoice.startCapture().then( ( val : CaptureAudioResult ) => 
{
    //Set in Process Flag false! (capture process finish!)
    this.inProcess = false;

    //Audio file is in blob:
    //val.audiomp3 (Blob value)

    //Optional -- Convert Mp3 audio to Audio Player
    this.player    = new Audio(URL.createObjectURL(val.audiomp3));
    //Optional -- Show a Audio player
    this.player.controls = true;

    //Optional -- add Audio player to div html
    this.divPlayer.appendChild(this.player);        
});
```
----------------------------------------------
----------------------------------------------
## <a name="CaptureFinger"></a>CaptureFinger
Capture Fingers

Example Layout HTML
```html
<p>Capture Fingers Demo</p>
<div id="divCaptura">
    <button id="btnStartCapture" (click)="initSDK()" value="InitSDK"
    class="btn btn-primary">1.Init SDK</button>

    <div [style]="sdisplay">
        <button id="btnStartCapture" (click)="startCapture()" value="Start Capture..."
            class="btn btn-primary">2. Capturre Fingers</button>
    </div>

    <div>
        <button id="btnDecodeBlob" (click)="decodeBlob()" value="decode Blob"
            class="btn btn-primary">3. Decode blob</button>
    </div>

    <div id="txtmsg">{{msgs}}</div>
    <div *ngIf="lstItems">
     <table style="width:100%;">
        <tr *ngFor="let item of lstItems">
            <td>                
                Finger {{item.fingerId}}   
            </td>
            <td>
                <img [src]='item.handName'>
           </td>
        </tr>
      </table>
    </div>
</div>

```
## CSS/Style
Capture fingers require this CSS Style items:
In Angular project add/import in styles.css

download from https://github.com/BytteDoc/BytteBrowserSDK/blob/main/finger-style.css

in your main CSS file, Link this file

```TypeScript
@import "finger-style.css";
...
```

## Assets
in your assets folder add this files
* 3cc8e488a00c7d64abb1d9d6ab43afe6.wasm
* 004a6983c2492a4a7553fd30e59406d2.wasm
* 91e19dba55a8cacf85d7d006f3d9e2d2.wasm
* 0625e28fcbdfc918b5f42ac1679c8255.wasm
* Images Folder

to get this files, download from https://github.com/BytteDoc/BytteBrowserSDK/blob/main/assets.zip


## Javascript/TypeScript Code  

```TypeScript
import { Component, OnInit } from '@angular/core';
import { CaptureEvents, CaptureFinger, FingerDetectionEnum } from '@bytte/byttebrowsersdk';
...  
//1. Declare CaptureFinger object
private oCapturaFin : CaptureFinger; //SDK Object  
private oEvents : CaptureEvents; //SDK Events
private bBlob : Blob; //Blob Response  
public  lstItems : FingerScore[]; //Array Fingers

```
Set FingerDetection type, the options are:
* L4F = Left 4 Fingers
* R4F = Right 4 Fingers 
* RIGHT_INDEX
* RIGHT_MIDDLE
* RIGHT_RING
* RIGHT_LITTLE
* LEFT_INDEX
* LEFT_MIDDLE
* LEFT_RING
* LEFT_LITTLE
* RIGHT_THUMB
* LEFT_THUMB

```TypeScript
//2. declare Constructor objects
const captureType = FingerDetectionEnum.R4F;
const wasmURL = '...'; //URL WASM Provided by Bytte
const timeOutNumber = 30000; //Time Out
const language = "es"; //Language es-en
const debug = false; //Debug - default false

//3. Generate a new object
this.oCapturaFin = new CaptureFinger(captureType, wasmURL, timeOutNumber, debug, language);
  
//4. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oCapturaFin.addCallBackStreamReady((streamReady: boolean, displayInfo?: string)  =>
{
    console.log("Stream Ready", streamReady);
    //Ready to Start!!

     this.msgs = "2. SDK Ready, Capture Fingers";      
});  

```
>5. Initialize SDK Finger Capture
This procedure load sdk elements in memory

```TypeScript
initSDK() 
{
    //Init Capture SDK     
    this.oCapturaFin.InitFingerCapture().then(() =>
        console.log("Init SDK") //SDK init successfully
).catch(err => 
    { 
        console.log("Error SDK", err) 
    });
}
```
> Call Capture function! 
> This procedure activate the back device Camera
> and Cavas object

> Start Capture!!
startCapture function return a promise.
startCapture initialize mobile phone camera and shows a square with instructions.
When promise return successfully, object return is a Blob JavaScript Object

```TypeScript
//6. Start Capture!!
...
startCapture(): void 
  {    
    this.oCapturaFin.startCapture().then((val: Blob) => 
    {
      this.bBlob = val;
      console.log("Blob OK");
      this.msgs = "3. Captura Lista";
    }).catch(err => {
      console.log(err);
      this.msgs = " ** Error " + err;
    });
  }

```
>9. Decode Blob
decodeBlob function convert the blob object in a Javascript Object with finger images

```TypeScript

//Decode blob
decodeBlob()
{
    if (this.bBlob) 
    {
            this.oCapturaFin.decodeBlob(this.bBlob)
        .then((fResult : FingerResult) => 
        {
          console.log("decodeBlob OK");
          console.log(fResult);
          this.msgs = "decode OK ";

          fResult.fingerScore.forEach(element => {
              element.handName = "data:image/png;base64," + element.image; 
          });

          //Asigno las imagenes
          this.lstItems = fResult.fingerScore;
        })
        .catch((err) => {
          console.log("decodeBlob ERROR ", err);
          this.msgs = "decode Error " + err;
        });
    }
  }
```
> DecodeBlob function return a FingerResult class object
this object contains images and information from the current captrure:

```TypeScript

import { FingerScore } from "./FingerScore";
export declare class FingerResult {
    score: number;
    fingerScore: FingerScore[];
    successOperation: boolean;
    message: string;
    constructor();
}

export declare class FingerScore {
    fingerId: number;
    score: number;
    image: string;
    wsq: string;
    fingerName: string;
    handName: string;
    width: number;
    height: number;
    constructor();
}
```

> DecodeBlob function return a FingerResult class object
this object contains images and information from the current captrure:
this is a example in JSON:
```TypeScript
{
    "score": 0,
    "fingerScore": [{
            "fingerId": 3,
            "score": 1,
            "image": "iVBORw0KGgo...", //Image ib base64 string encoding           
            "fingerName": "MIDDLE",
            "handName": "RIGHT",
            "width": 269,
            "height": 444
        }, {
            "fingerId": 4,
            "score": 1,
            "image": "iVBORw0KGgoAAA...", //Image ib base64 string encoding                      
            "fingerName": "RING",
            "handName": "RIGHT",
            "width": 252,
            "height": 423
        }, {
            "fingerId": 2,
            "score": 1,
            "image": "iVBORw0KGg....",   //Image ib base64 string encoding                     
            "fingerName": "INDEX",
            "handName": "RIGHT",
            "width": 269,
            "height": 402
        }, {
            "fingerId": 5,
            "score": 1,
            "image": "iVBORw0...",  //Image ib base64 string encoding           
            "fingerName": "LITTLE",
            "handName": "RIGHT",
            "width": 247,
            "height": 396
        }
    ],
    "successOperation": true,
    "message": "FEEDBACK_CAPTURED"
}
```
> Return types:

* When successOperation = true, capture was finished correctly
* When successOperation = false, capture was with error, the "message" variable contains this detail
* fingerId: is the finger number (1 .. 10)
* image: string base64 image in PNG format
* fingerName: finger captured name: INDEX, MIDDLE, RING, LITTLE, THUMB
* handName: hand captured: RIGHT or LEFT

----------------------------------------------
----------------------------------------------
## <a name="CaptureFace"></a>CaptureFace
* Capture Face

Example Layout HTML
```html
<p>Capture Face Demo</p>
<div id="divCaptura">
    <button id="btnStartCapture" (click)="initSDK()" value="InitSDK"
    class="btn btn-primary">1.Init SDK</button>

    <div [style]="sdisplay">
        <button id="btnStartCapture" (click)="startCapture()" value="Start Capture..."
            class="btn btn-primary">2. Capture Face</button>
    </div>
    <div>
        <button id="btnDecodeBlob" (click)="decodeBlob()" value="decode Blob"
            class="btn btn-primary">3. Decode blob</button>
    </div>
    <div id="txtmsg">{{msgs}}</div>
    <div *ngIf="sImage">
        <table>
           <tr>
               <td>                
                   Face
               </td>
               <td>
                   <img style="width:40%;" [src]='sImage'>
              </td>
           </tr>
         </table>
       </div>
</div>

```
## CSS/Style
Capture face require this CSS Style items:
In Angular project add/import in styles.css

download from https://github.com/BytteDoc/BytteBrowserSDK/blob/main/face-style.css

in your main CSS file, Link this file

```TypeScript
@import "face-style.css";
...
```

## Assets
in your assets folder add this files
* 2d57157acbcac85f72ed68bceb90ffa3.wasm
* Images Folder

to get this files, download from https://github.com/BytteDoc/BytteBrowserSDK/blob/main/assets.zip


## Javascript/TypeScript Code  

```TypeScript
import { Component, OnInit } from '@angular/core';
import * as BytteSDK from "@bytte/byttebrowsersdk";
import { FaceResult } from '@bytte/byttebrowsersdk/types/DataTypes/FaceResult';
...  
//1. Declare Capture Face object
private oCapturaFace: BytteSDK.CaptureFace; //SDK Object  
private bBlob : Blob; //Blob Response  

//2. declare Constructor objects
const wasmURL = '...';          //URL WASM Provided by Bytte
const timeOutNumber = 30000;    //Time Out
const language = "es";          //Language es-en
const debug = false;            //Debug - default false

//3. Generate a new object
this.oCapturaFace = new BytteSDK.CaptureFace(wasmURL, timeOutNumber, debug, language);    
  
//4. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oCapturaFace.addCallBackStreamReady((streamReady: boolean, displayInfo?: string) => {
      console.log("streamReady", streamReady);
      this.msgs = "2. SDK Ready, Capture Face";            
});

```
>5. Initialize SDK Face Capture
This procedure load sdk elements in memory

```TypeScript
initSDK() 
{
    //Init Capture SDK     
    this.oCapturaFace.InitFaceCapture().then(() => {
      console.log("Init SDK");
    }
    ).catch(err => {
      console.log("Error SDK", err)
      this.msgs = "Init Error " + err;
    });
  }
```
> Call Capture function! 

> This procedure activate the back device Camera and Cavas object

> Start Capture!!
startCapture function return a promise.
startCapture initialize mobile phone camera and shows a square with instructions.
When promise return successfully, object return is a Blob JavaScript Object

```TypeScript
//6. Start Capture!!
...
  startCapture(): void {
    this.oCapturaFace.startCapture().then((val: Blob) => {
      this.bBlob = val;
      console.log("Blob OK");
      this.msgs = "3. Capture ready..";
    }).catch(err => {
      console.log(err);
      this.msgs = " ** Error " + err;
    });
  }

```
>9. Decode Blob
decodeBlob function convert the blob object in a Javascript Object with the face image

```TypeScript

//Decode blob
 decodeBlob() {
    if (this.bBlob) 
    {
      this.msgs = "Process a Blob..."; 
      this.oCapturaFace.decodeBlob(this.bBlob)
        .then((fResult: FaceResult) => {
          if (fResult.successOperation) {
            console.log("decode Blob OK");
            console.log(fResult);
            this.msgs = "decode Ok ";
            this.sImage = "data:image/png;base64," + fResult.faceScore.image;
          }
          else {
            this.msgs = "Error in process! " + fResult.message;
          }

        })
        .catch((err) => {
          console.log("decodeBlob ERROR ", err);
          this.msgs = "decode Error " + err;
        });
    }
  }
```
> DecodeBlob function return a FaceResult class object
this object contains images and information from the current captrure:

```TypeScript
import { FaceScore } from "./FaceScore";
export declare class FaceResult {
    score: number;
    faceScore: FaceScore;
    successOperation: boolean;
    message: string;
    constructor();
}
...
...

import { FaceQuality } from "./FaceQuality";
export declare class FaceScore {
    faceId: string;
    score: number[];
    image: string;
    width: number;
    height: number;
    bitsPerPixel: number;
    channels: number;
    resolution: string;
    quality: FaceQuality;
    constructor();
}

...
...
export declare class FaceQuality {
    eyesStatus: string;
    eyesOpenMetric: number;
    headPoseCheck: boolean;
    qualityMetric: number;
    constructor();
}
```

> DecodeBlob function return a FingerResult class object
this object contains images and information from the current captrure:
this is a example in JSON:
```TypeScript
{
    "faceScore": {
        "faceId": "ccb9e28a-454e-a74c-adff-5e5f842412971634162085145",
        "score": [1],
        "image": "iVBORw0...",
        "width": 540,
        "height": 720,
        "bitsPerPixel": 8,
        "channels": 3,
        "resolution": "540x720x3",
        "quality": {
            "eyesStatus": "OPEN",
            "eyesOpenMetric": 79.9,
            "headPoseCheck": true,
            "qualityMetric": 86.72
        }
    },
    "successOperation": true,
    "message": "FEEDBACK_CAPTURED"
}

```
> Return types:

* When successOperation = true, capture was finished correctly
* When successOperation = false, capture was with error, the "message" variable contains this detail 
* image: string base64 image in PNG format
* score: percentual value (0 - 1)
* others: image details information
