# Bytte Browser SDK

![N|Solid](https://www.bytte.com.co/pagweb/wp-content/uploads/2019/05/by-casb-logo.png)

**Bytte Browser SDK** is an SDK that provides the functionality to perform different captures (documents, face, voice) from a web browser 

## Features

 - [Colombia Document Front Capture](#CaptureCOBackDocument).
 - [Colombia Document Back Capture](#CaptureCOFrontDocument).
 - [Colombia Digital Document Front Capture](#CaptureCODigitalFrontDocument).
 - [Colombia Digital Document Back Capture](#CaptureCODigitalBackDocument)
 - [Face Capture](#CaptureFace)
 - [Voice Capture](#VoiceCapture)
 - [Capture Result object](#CaptureDocumentoResult)
 - Finger Capture - Coming soon...

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
npm install @bytte/byttebrowsersdk@1.0.8
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
## <a name="CaptureFace"></a>CaptureFace
Capture Face Component

Example Layout HTML
```html
<p>capture face</p>

<div id="screen-start" class="hidden">    
    <button id="btnStartCapture" (click)="startCapture()" value="Start Capture..." class="btn btn-primary">Capture</button>
</div>
 
    <!-- Control -->
    <div class="dvContainerbase hidden" id="dvContainerbase">
        <div id="document_picker" class="document_pickerbase">
            <div id="pageContainerdiv">
                <div class="cameraContainer">
                    <video id="user-video" autoplay playsinline class="user-video"></video>
                    <canvas id="overlay" class="overlay"></canvas>
                    <img id="mask" class="overlay" />                    
                </div>
            </div>
        </div>
    </div> 
    <!-- End Control -->

    <div>
        <!-- Callback info -->
        {{processInfo}}
    </div>

<div style="display:block;"> 
    <div id="dvResult">
        <img id="imgResult" [src]="imSrc" />        
    </div>    
</div>
<label id="lblTest">--</label><br />

<!-- !!!!Important!!!!! -->
<!-- Div with canvas hidden required -->
<div hidden>
    <canvas></canvas>
    <canvas id="overlayResult"></canvas>
    <canvas id="canvasmirror"></canvas>
    <div id="seccion_resultado" hidden>
        <input id="code-result" />
    </div>
</div>
```

## Javascript/TypeScript Code  

```TypeScript
import * as BytteSDK from "@bytte/byttebrowsersdk";
import { BaseReturn, CaptureDocumentoResult } from '@bytte/byttebrowsersdk';
...
...  
//1. Declare CaptureFace object
private oSDKCapturaRostro: BytteSDK.CaptureFace;

//2. Get HTML reference objects
private cameraFeed: HTMLVideoElement;
private cameraFeedback: HTMLCanvasElement;
private canvasMirror: HTMLCanvasElement;
private dvContainerbase: HTMLDivElement;
private sWeightsPath: string;
private oFaceMask: HTMLImageElement;
...
...
this.cameraFeed = document.getElementById("user-video") as HTMLVideoElement;
this.cameraFeedback = document.getElementById("overlay") as HTMLCanvasElement;
this.canvasMirror = document.getElementById("canvasmirror") as HTMLCanvasElement;
this.dvContainerbase = document.getElementById("dvContainerbase") as HTMLDivElement;
this.oImgResult = document.getElementById("imgResult") as HTMLImageElement;
this.oFaceMask = document.getElementById("mask") as HTMLImageElement;
 
//3. Instance SDK
this.oSDKCapturaRostro =  new  BytteSDK.CaptureFace(this.cameraFeed,
this.cameraFeedback,
this.canvasMirror,
this.dvContainerbase,
this.oFaceMask,
this.sWeightsPath,
this.faceMaskPath,
true,  //Return base64 Image for test only!!
false, //Procecss FaceLandmarks - always false
this.sCamInfo);

```
> Return image Base64 string **IS NOT RECOMENDED**, use only for test
> SDK return a blob object, this contains the image

```TypeScript
//4. Set Time Out (example 30 segs)
this.oSDKCapturaRostro.setTimeOut(30  *  1000);  

//5. Initialize SDK Face Capture
//Return Promise void
this.oSDKCapturaRostro.InitFaceCapture().then((x) => {
         console.log("Init Face Capture OK!!");
});
  
//6. add Callback Stream Ready
// When the SDK prepares the cameras (this process can take a long time),
// the user could see, for example, a waiting image.....
// When done, this callback will be called
this.oSDKCapturaFrenteDoc.addCallBackStreamReady((streamReady: boolean, displayInfo?: string)  =>
{
    console.log("Stream Ready", streamReady);
    //Ready to Start!!

    /* if display Info return, save this string */
    if (displayInfo) {
        this.sCamInfo = displayInfo;
    }
});  

//6. add Callback Process info
this.oSDKCapturaRostro.addCallFaceDetectionErrors((pipelineResults: PipelineResults) => {
         this.processInfo = "";
         if (pipelineResults.FACE_ANGLE_TOO_LARGE) {
            this.processInfo += " FACE ANGLE TOO LARGE";
         }
         if (pipelineResults.FACE_CLOSE_TO_BORDER) {
            this.processInfo += " FACE CLOSE TO BORDER";
         }
         if (pipelineResults.FACE_NOT_FOUND) {
            this.processInfo += " FACE NOT FOUND";
         }
         if (pipelineResults.FACE_TOO_SMALL) {
            this.processInfo += " FACE TOO SMALL";
         }
         if (pipelineResults.PROBABILITY_TOO_SMALL) {
            this.processInfo += " PROBABILITY TOO SMALL";
         }
         if (pipelineResults.TOO_MANY_FACES) {
            this.processInfo += " TOO MANY FACES";
         }
      });

```
> Call Capture function! 
> This procedure activate the back device Camera
> and Cavas object

```TypeScript
//7. Start Capture!!
...
this.oSDKCapturaRostro.startCapture().then(oCapturaDocumentoResult => 
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
    /*
    --> In documentImageBase64 return Face Image Base64String 
    */

    //if in the constructor, flag return imageBase64 is true, 
    //**return imageBase64 IS NOT RECOMENDED**
    documentImageBase64 field contains a image
    this.oImgResult.src = "data:image/png;base64," + oDocumentoResult.documentImageBase64;  
    
    this.processInfo = "Capture end success";
    //Send instruction to SDK to End Capture
    //End stream and free memory resources
    this.oSDKCapturaRostro.endCapture().then(() => {
        console.log("Fin Captura");
        })
}

//Handle Error
async ProcesaCaptureError(oDocumentoResult: CaptureDocumentoResult) {
    //Show error in div..  
    this.documentResult = oDocumentoResult;
    this.mensaje = oDocumentoResult.messaje;
    console.log("Error Capture");
    console.log(oDocumentoResult);
    this.processInfo = "Capture Error";
} 

/* if you need a clean image of base64 memory
   when the mobile device is low on memory 
*/
cleanMem() {
    this.oImgResult.src = "";
}

```

## --------------------------------------------------
## <a name="CaptureDocumentoResult"></a>CaptureDocumentoResult Object Return
When process finish correctly, a **CaptureDocumentoResult** return:

```TypeScript
//Blob With a image
documentImageBlob: Blob | null;

//Image in Base64 String Format
//if in constructor, ReturnDocumentBase64Image = true
documentImageBase64: string;

//Return Base64String BarCode PDF417 information
barCodeBase64: string;

//First Name
firstName: string;

//Last Name
lastName: string;

//Full Name
fullName: string;

//Document Number
documentNumber: string;

//Status - if true, capture success
status: boolean;

//If error, show exception details
messaje: string;
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

