<template>
  <div
    class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4"
  >
    <h1 class="text-2xl font-bold mb-4">Extract Hidden Message</h1>

    <!-- Input field for uploading the image -->
    <div class="mb-4">
      <label class="block mb-2">Upload Image with Hidden Message:</label>
      <input
        type="file"
        accept="image/*"
        @change="handleImageUpload"
        class="p-2 border border-gray-300 rounded-lg cursor-pointer"
      />
    </div>

    <button
      @click="extractMessage"
      :disabled="!imageSrc"
      class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
    >
      Extract Message
    </button>

    <div v-if="hiddenMessage" class="mt-4">
      <h2 class="text-xl font-semibold mb-2">Hidden Message:</h2>
      <p class="text-lg">{{ hiddenMessage }}</p>
    </div>

    <!-- Navigation button -->
    <div class="mt-4">
      <button
        @click="navigateToHide"
        class="px-4 py-2 bg-gray-500 text-white rounded-lg hover:bg-gray-600"
      >
        Go to Hide Page
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { useRouter } from "#app";

const imageSrc = ref(null);
const hiddenMessage = ref("");
const router = useRouter();

function handleImageUpload(event) {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      imageSrc.value = e.target.result;
    };
    reader.readAsDataURL(file);
  }
}

function extractMessage() {
  if (!imageSrc.value) return;

  const img = new Image();
  img.onload = () => {
    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;

    // Extract the message
    let messageBits = "";
    for (let i = 0; i < data.length; i += 4) {
      messageBits += (data[i] & 1).toString();
      if (messageBits.length % 8 === 0) {
        const byte = messageBits.slice(-8);
        if (byte === "00000000") {
          break; // Null terminator
        }
      }
    }

    // Convert bits to string
    let message = "";
    for (let i = 0; i < messageBits.length; i += 8) {
      const byte = messageBits.substr(i, 8);
      if (byte === "00000000") break; // Null terminator
      message += String.fromCharCode(parseInt(byte, 2));
    }

    hiddenMessage.value = message;
  };
  img.src = imageSrc.value;
}

function navigateToHide() {
  router.push("/hide");
}
</script>

<style scoped>
/* Add any additional styles if needed */
</style>
