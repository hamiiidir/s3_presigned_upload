<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    
  </div>
</template>

<script setup>
  const msg = 'Hello Hamid'
  const { S3Client, ListObjectsCommand, ListBucketsCommand } = require('@aws-sdk/client-s3');

  const s3 = new S3Client({
      region: 'default',
      endpoint: '**********************',
      credentials: {
          accessKeyId: '**********************',
          secretAccessKey: '************************',
      },
  });

  const listObjects = async (bucket) => {
      try {
          const response = await s3.send(
              new ListObjectsCommand({
                  Bucket: bucket,
              })
          )
          console.log('Success', response);
      } catch (err) {
          console.log('Error', err);
      }
  };

  const listBuckets = async () => {
    try {
        const data = await s3.send(new ListBucketsCommand({}))
        console.log('Success', data.Buckets)
    } catch (err) {
        console.log('Error', err);
    }
  }

  listObjects('new-box')
  listBuckets()
  
</script>

