<template>
    <div>
        <video ref="video" autoplay playsinline></video>
    </div>
</template>
  
<script>
export default {
    name: 'LiveCamera',
    mounted() {
        this.initCamera();
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
        }
    }
};
</script>
  
<style>
video {
    width: 100%;
    /* Adjust as necessary */
}
</style>
  