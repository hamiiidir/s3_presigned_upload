<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    
  </div>
</template>

<script setup>
  const msg = 'Hello Hamid'
  const { S3Client, ListBucketsCommand } = require('@aws-sdk/client-s3');

  const s3 = new S3Client({
      region: 'default',
      endpoint: 'https://s3.ir-thr-at1.arvanstorage.com',
      credentials: {
          accessKeyId: '4be0aaa0-584c-4abf-8627-74efce3e6fc3',
          secretAccessKey: '277a38c8f71e2b887380426eeca29aeb33be7ce6e43c4fc34dbf7fb1e2dada0d',
      },
  });

  // const listObjects = async (bucket) => {
  //     try {
  //         const response = await s3.send(
  //             new ListObjectsCommand({
  //                 Bucket: bucket,
  //             })
  //         )
  //         console.log('Success', response);
  //     } catch (err) {
  //         console.log('Error', err);
  //     }
  // };

  const listBuckets = async () => {
    try {
        const data = await s3.send(new ListBucketsCommand({}))
        console.log('Success', data.Buckets)
    } catch (err) {
        console.log('Error', err);
    }
  }

  // listObjects('new-box')
  listBuckets()
  
</script>

