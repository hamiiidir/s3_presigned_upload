<template>
    <div class="flex justify-center items-center overflow-x-hidden h-screen border-slate-200">
        <div @dragover="dragOverHandler($event)" @dragleave="dragLeaveHandler($event)" @drop="dropHandler($event)" class="w-1/2 h-1/2 p-10 scale-100 transition-all duration-150 border-8 border-dashed">
            
        </div>
        <input ref="fileInput" @change.prevent="uploadFile($event.target.files)" type="file" id="file-upload-input" name="files" multiple hidden>
    </div>  
</template>

<script setup>
const { S3Client } = require('@aws-sdk/client-s3');

const s3 = new S3Client({
    region: 'default',
    endpoint: 'https://s3.ir-thr-at1.arvanstorage.com',
    credentials: {
        accessKeyId: '4be0aaa0-584c-4abf-8627-74efce3e6fc3',
        secretAccessKey: '277a38c8f71e2b887380426eeca29aeb33be7ce6e43c4fc34dbf7fb1e2dada0d',
    },
});

onMounted(() => {
    console.log('Hello')
    console.log(s3)
})

</script>


