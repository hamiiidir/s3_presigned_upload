<template>
    <div class="flex justify-center items-center overflow-x-hidden h-screen border-slate-200">
        <div @dragover="dragOverHandler($event)" @dragleave="dragLeaveHandler($event)" @drop="dropHandler($event)" class="w-1/2 h-1/2 p-10 scale-100 transition-all duration-150 border-8 border-dashed">
            
        </div>
        <input ref="fileInput" @change.prevent="uploadFile($event.target.files)" type="file" id="file-upload-input" name="files" multiple hidden>
    </div>  
</template>

<script setup>
import { ref } from "@vue/reactivity";

const { S3Client, PutBucketCorsCommand } = require('@aws-sdk/client-s3');
const { createPresignedPost } = require('@aws-sdk/s3-presigned-post');

const fileInput = ref() 
const uploadedFiles = ref()

const accessKeyId = '********************************'
const secretAccessKey = '**************************************'
const bucket = 'my-bucket'

function dragOverHandler(ev) {
    console.log('File(s) in drop zone');
    
    ev.preventDefault();

}

function dragLeaveHandler(ev){
    ev.preventDefault();

}

function dropHandler(ev) {
    console.log('File(s) dropped');

    ev.preventDefault();


    uploadFile(ev.dataTransfer.files)

}

async function uploadToS3(fileToUpload){
    // Create an S3 client service object
    const s3 = new S3Client({
        region: 'default',
        endpoint: 'https://s3.ir-thr-at1.arvanstorage.com',
        credentials: {
            accessKeyId: accessKeyId,
            secretAccessKey: secretAccessKey,
        },
    });

    console.log(fileToUpload);

    // const file = fileToUpload;
    const Bucket = bucket;
    const Key = fileToUpload.name;
    const Fields = {
        acl: 'public-read',
    };

    const Conditions = [
        { acl: 'public-read' },
        { bucket: Bucket },
        // ['starts-with', '$key', 'user/'],
    ];

    let signatureResponse
    let signatureUrl
    const run = async () => {
        const { url, fields } = await createPresignedPost(s3, {
            Bucket,
            Key,
            Conditions,
            Fields,
            Expires: 600, //Seconds before the presigned post expires. 3600 by default.
        });

        console.log('url', url);
        console.log('fields', fields);
        signatureUrl = url
        signatureResponse = fields
    };

    
    const run2 = async () => {
        try {
            const cors = {
                Bucket: Bucket, // REQUIRED
                CORSConfiguration: {
                    // REQUIRED
                    CORSRules: [
                        // REQUIRED
                        {
                            AllowedHeaders: ['*'],
                            AllowedMethods: ['POST'], // REQUIRED
                            AllowedOrigins: ['*'], // REQUIRED
                        },
                    ],
                },
            };
            const response = await s3.send(new PutBucketCorsCommand(cors));
            console.log('Success', response);
        } catch (err) {
            console.log('Error', err);
        }
    };  

    function run3(fileToUpload) {
        const formData = new FormData();
        formData.append("key", signatureResponse['key']);
        formData.append("X-Amz-Algorithm", signatureResponse['X-Amz-Algorithm']);
        formData.append("X-Amz-Credential", signatureResponse['X-Amz-Credential']);
        formData.append("X-Amz-Date", signatureResponse['X-Amz-Date']);
        formData.append("acl", signatureResponse['acl']);
        formData.append("policy", signatureResponse['Policy']);
        formData.append("X-Amz-Signature", signatureResponse['X-Amz-Signature']);
        formData.append("file", fileToUpload);

        var xhr = new XMLHttpRequest();

        xhr.upload.addEventListener("progress", function (e) {
            if (e.lengthComputable) {
                console.log(e.loaded / e.total);
            }
        });

        xhr.upload.addEventListener("load", function () {
            console.log("uploaded");
        });

        for (const [key, value] of formData) {
            console.log(`${key}: ${value}`);
        }

        xhr.open("POST", signatureUrl);
        xhr.send(formData);
    }    

    await run()
    await run2()
    await run3(fileToUpload)


}

function uploadFile(files){

    uploadedFiles.value = Array.from(files)

    fileInput.value.value = null

    uploadToS3(uploadedFiles.value[0])

}

</script>


