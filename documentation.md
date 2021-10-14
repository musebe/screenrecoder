### Cloudinary Screen Recorder
## Background
Screen recorder is a software that turns screen output to a video . It’s usefully applicable in use cases such as making videos of screen sequences to log results for troubleshooting, narration during capture and creating software tutorials. 

## Prerequisites
Basic understanding of Javascript and React

## Setting Up the Sample Project
Using your terminal, generate a Next.js by using the create-next-app CLI:

`npx create-next-app screenrec`

Go to your project directory :

`cd screenrec`

Install the required dependencies. We will use materialize to style our app’s buttons, cloudinary for our app’s cloud storage and dotenv to handle cloudinary environment variables:

`npm install cloudinary dotenv @materialize-ui/core`

## Cloudinary Setup

We begin with setting up a free cloudinary account here. In your account’s dashboard, you will receive environment keys to use in integrating cloudinary to your project.

Create a file named ‘.env’ in your project root directory and paste in the following:

`CLOUDINARY_API_KEY = `

`CLOUDINARY_API_SECRET=`

Complete the above with information from your cloudinary account.

Go to the ‘pages/api’ directory and create a folder called ‘utils’. Inside the folder create a file named ‘cloudinary’. And paste the following code 
 ```
require('dotenv').config();
const cloudinary = require('cloudinary');
cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
    upload_preset: process.env.CLOUDINARY_UPLOAD_PRESET,
});

module.exports = { cloudinary };
```

Our last cloudinary setup phase will involve setting up our project’s backend. Inside the api directory , create a file named upload.js. We’ll use this component to upload  the recorded 
```
var cloudinary = require("cloudinary").v2;

cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
    upload_preset: process.env.CLOUDINARY_UPLOAD_PRESET,
});
export const config = {
    api: {
        bodyParser: {
            sizeLimit: "20mb",
        },
    },
};

export default async function handler(req, res) {
    let uploaded_url = ""
    const fileStr = req.body.data;

    if (req.method === "POST") {
        try {
            const uploadedResponse = await cloudinary.uploader.upload_large(
                fileStr,
                {
                    resource_type: "video",
                    chunk_size: 6000000,
                }
            );
            console.log("uploaded_url", uploadedResponse.secure_url)
            uploaded_url = uploadedResponse.secure_url
        } catch (error) {
            res.status(500).json({ error: "Something wrong" });
        }
        res.status(200).json({ data : uploaded_url });
    }
}
```

With our backend complete , let us create our screen recorder.
Create a folder named ‘components’ in your project root directory and inside it create a file named ‘record.js’. 

## Recording video output
To do this we require the following:
use-screen-record which is a react hook for recording screen using MediaStream APIs.
Materialize-ui used to design our UI buttons
Include the above using the following imports:

```
import React, { useRef, useState } from "react";

import useScreenRecorder from "use-screen-recorder";

import Button from "@material-ui/core/Button"

}
```

Create a function ‘Recorder’ that will use the use-screen-record react hook to record screen using MediaStream APIs.

```
function Recorder() {
    const videoRef = useRef();
    const [link, setLink] = useState();

    let arr = []

    const {
        blobUrl,
        pauseRecording,
        resetRecording,
        resumeRecording,
        startRecording,
        status,
        stopRecording,
    } = useScreenRecorder({ audio: true });
 
    return (
        <div>
            <h1 className='elegantshadow'>Screen Recorder</h1>
            <div className='status'>
                Status: {status}<br /><br />
                Video Link: {link || "Waiting..."}
            </div>
            <div>
                <video
                    ref={videoRef}
                    src={blobUrl}
                    controls
                    autoPlay
                />
            </div>
            <div className='buttons'>
                {(status === "idle" || status === "error") && (
                    <Button onClick={startRecording} variant='contained' color='primary' >Start recording</Button>
                )}
                {(status === "recording" || status === "paused") && (
                    <Button onClick={stopRecording} variant='contained' color='primary'>Stop recording</Button>
                )}{' '}
                {(status === "recording" || status === "paused") && (
                    <Button
                        onClick={() =>
                            status === "paused" ? resumeRecording() : pauseRecording()
                        }
                        variant='contained' color='primary'
                    >
                        {status === "paused" ? "Resume recording" : "Pause recording"}
                    </Button>
                )}
                {status === "stopped" && (
                    <Button
                        onClick={() => {
                            resetRecording();
                            videoRef.current.load();
                        }}
                        variant='contained' color='primary'
                    >
                        Reset recording
                    </Button>

                )}{' '}
                {status === "stopped" && (
                    <Button
                        onClick={handleVideo}
                        variant='contained' color='primary'
                    >
                        Upload Recording
                    </Button>

                )}
            </div>
        </div>
    )
} export default Recorder;
}
```

Below the function create another function handleVideo to retrieve the recorded video from the react hook and convert it to base64 and pass the encoded video to the handleUpload function.  

```
async function handleVideo() {
    let blob = await fetch(blobUrl).then(r => r.blob());
    var reader = new FileReader();
    reader.readAsDataURL(blob);
    reader.onloadend = function () {
        var base64data = reader.result;
        handleUpload(base64data);
    }
    
}
```

The handleUpload function uploads the recorded video to the backend where its delivered to cloudinary and returns a response containing the respective cloudinary link used to store the video.

```
 async function handleUpload(video) {
        try {
            fetch("/api/upload", {
                method: "POST",
                body: JSON.stringify({ data: video }),
                headers: { "Content-Type": "application/json" },
            })
                .then((response) => {
                    console.log("response", response.status)
                    response.json().then((data) => {
                        arr.push(data)
                        setLink(arr[0].data)
                    });
                });
        } catch (error) {
            console.error(error);
        }
    }
}
```

Thats it! You can now recorde your screen as video and upload it to cloudinary. Your video link will automatically appear in the link tab above the video