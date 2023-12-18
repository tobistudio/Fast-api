<template>
    <f7-page name="age-gender-recognition">
        <f7-navbar title="Age and Gender Recognition">
            <f7-nav-right>
                <f7-button style="width: 100%;" @click="initCamera">Open Camera</f7-button>
            </f7-nav-right>
        </f7-navbar>

        <!-- <f7-input type="file" @change="handleFileChange" accept="video/*" /> -->
        <f7-block>
            <div class="canvas-container">
                <video ref="video" :src="videoPreviewUrl" autoplay muted loop playsinline
                    :onloadedmetadata="detectFaces"></video>
                <canvas ref="canvas"></canvas>
            </div>
        </f7-block>
    </f7-page>
</template>
  
<script>
import * as faceapi from 'face-api.js';

export default {
    mounted() {
        this.loadModels();
    },
    data() {
        return {
            ImageSrc: '',
            videoPreviewUrl: '/animation.mp4'
        };
    },
    methods: {
        async initCamera() {
            try {
                const stream = await this.startCamera();
                this.$refs.video.srcObject = stream;
            } catch (error) {
                console.error('Error initializing camera', error);
            }
        },

        async startCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                return stream;
            } catch (error) {
                console.error('Error accessing the camera', error);
                throw error;
            }
        },

        async loadModels() {
            try {
                await Promise.all([
                    faceapi.nets.tinyFaceDetector.loadFromUri('/models'),
                    faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
                    faceapi.nets.ageGenderNet.loadFromUri('/models')
                ]);
                console.log('Models loaded successfully!');
            } catch (error) {
                console.error('Error loading models', error);
            }
        },

        async detectFaces() {
            let canvas = this.$refs.canvas;
            const video = this.$refs.video;

            const displaySize = { width: video.videoWidth, height: video.videoHeight };
            faceapi.matchDimensions(canvas, displaySize);

            const onPlay = async () => {

                if (video.paused || video.ended)
                    return setTimeout(() => onPlay())

                const detections = await faceapi.detectAllFaces(
                    video,
                    new faceapi.SsdMobilenetv1Options()
                ).withAgeAndGender();

                const resizedDetections = faceapi.resizeResults(detections, displaySize);
                canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);

                faceapi.draw.drawDetections(canvas, resizedDetections);

                const canvasWidth = canvas.width;

                // Let's say you want the font size to be 5% of the canvas width
                const fontSizePercentage = 3;
                const fontSize = canvasWidth * (fontSizePercentage / 100);

                // Log age and gender for demonstration
                detections.forEach(detection => {
                    const { age, gender } = detection;
                    new faceapi.draw.DrawTextField(
                        // [`${gender}`],
                        [`${parseInt(age)} years`, `${gender}`],

                        detection.detection.box.bottomLeft,
                        {
                            fontSize: fontSize, // Set your desired font size here
                        }
                    ).draw(canvas);
                });
                setTimeout(() => onPlay())
            };
            video.addEventListener('play', onPlay)
        }
    }
};
</script>
  
<style>
.canvas-container {
    position: relative;
}

.canvas-container img {
    width: 100%;
}

.canvas-container video {
    width: 100%
}

/* Add your custom styles here */
.canvas-container canvas {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
}
</style>
  