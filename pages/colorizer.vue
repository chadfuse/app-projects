<template>
  <div>
    <v-container fluid class="container--fluid fill-height">
      <v-row class="align-middle">
        <v-col cols="12" md="6" sm="12">
          <h1 class="text--white">Photo Colorization</h1>
          <p>
            This small project that utilizes the power of artificial intelligence for photo colorization! this project enables you to upload black and white photos and instantly transform them into vibrant, full-color images. Simply select a black and white photo using the provided upload button, and the AI algorithm will automatically generate a colored version of the image. 
          </p>
          <input type="file" @change="handleImageUpload" class="custom-file-input"/>
        </v-col>
        <v-col cols="12" md="6" sm="12" class="d-flex align-center justify-center">
          <div v-if="processing">
            <progress max="100" :value="progress"></progress>
            <p>Processing image...</p>
          </div>
          <div v-if="coloredImage">
            <img :src="coloredImage" width="100%" class="outimage" />
          </div>
          <div v-if="processedImage">
            <img :src="processedImage" width="100%" class="outimage" />
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>
export default {
  data() {
    return {
      processing: false,
      progress: 0,
      coloredImage: null,
      processedImage: null,
      taskId: null,
    };
  },
   head: {
    link: [
      { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
      { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: true },
      { href: 'https://fonts.googleapis.com/css2?family=Quicksand:wght@300;400;500;600;700&display=swap', rel: 'stylesheet' },
    ],
  },
  methods: {
    async handleImageUpload(event) {
      const file = event.target.files[0];
      const formData = new FormData();
      formData.append("file", file);
      formData.append("sync", "0");

      this.processing = true;

      try {
        const response = await fetch("https://techhk.aoscdn.com/api/tasks/visual/colorization", {
          method: "POST",
          headers: {
            "x-api-key": "wx0jof0r7yyyxf1u6",
          },
          body: formData,
        });

        if (response.status === 200) {
          const result = await response.json();
          this.taskId = result.data.task_id;
        } else {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
      } catch (error) {
        console.error(error);
      }

      this.processing = false;

      // Start polling for the task status
      this.pollTaskStatus();
    },
    async pollTaskStatus() {
      while (true) {
        try {
          const response = await fetch(`https://techhk.aoscdn.com/api/tasks/visual/colorization/${this.taskId}`, {
            headers: {
              "x-api-key": "wx0jof0r7yyyxf1u6",
            },
          });

          if (response.status === 200) {
            const result = await response.json();
            const state = result.data.state;

            if (state === 1) {
              // Colorization is complete
              const imageUrl = result.data.image;
              const processedResponse = await fetch(imageUrl);
              const processedResult = await processedResponse.blob();
              this.processedImage = URL.createObjectURL(processedResult);

              break; // Exit the polling loop
            } else if (state < 0) {
              // Colorization failed
              console.error("Colorization failed: state", state);
              break; // Exit the polling loop
            } else {
              // Colorization is still in progress
              this.progress = result.data.progress;
            }
          } else {
            throw new Error(`HTTP error! Status: ${response.status}`);
          }

          // Poll again after 1 second
          await new Promise((resolve) => setTimeout(resolve, 1000));
        } catch (error) {
          console.error(error);
          break; // Exit the polling loop
        }
      }
    },
  },
};

</script>
<style scoped>
h1, h2, h3, p, input {
  font-family: 'Quicksand', sans-serif !important;
}
.custom-file-input::-webkit-file-upload-button {
  visibility: hidden;
}
.custom-file-input::before {
  content: 'Select Image';
    display: inline-block;
    border-radius: 3px;
    padding: 10px;
    outline: none;
    white-space: nowrap;
    -webkit-user-select: none;
    cursor: pointer;
    font-weight: 700;
    font-size: 10pt;
    background: linear-gradient(90deg, #00C9FF 0%, #92FE9D 100%);
}
.custom-file-input:active::before {
  background: -webkit-linear-gradient(top, #e3e3e3, #f9f9f9);
}

.outimage {
  max-width: 400px !important;
  border: 10px solid white;
  border-radius: 10px;
}
h1{
  font-size: 50px;
}
</style>
