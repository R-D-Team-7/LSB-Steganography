<template>
  <div
    class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4"
  >
    <h1 class="text-2xl font-bold mb-4">LSB Steganography</h1>

    <!-- Input fields -->
    <div class="mb-4">
      <label class="block mb-2">Upload Image:</label>
      <input
        type="file"
        accept="image/*"
        @change="handleImageUpload"
        class="p-2 border border-gray-300 rounded-lg cursor-pointer"
      />
    </div>

    <div class="mb-4">
      <label class="block mb-2">Enter Secret Message:</label>
      <input
        type="text"
        v-model="secretMessage"
        class="p-2 border border-gray-300 rounded-lg"
      />
    </div>

    <button
      @click="processImage"
      :disabled="!imageSrc || !secretMessage"
      class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
    >
      Encode Message
    </button>

    <div v-if="outputImage" class="mt-4">
      <h2 class="text-xl font-semibold mb-2">Output Image:</h2>
      <img
        :src="outputImage"
        alt="Output Image"
        class="w-full max-w-md rounded-lg shadow-lg"
      />
      <a :href="outputImage" download="stego_image.png">
        <button
          class="mt-2 px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600"
        >
          Download Image
        </button>
      </a>
    </div>

    <!-- Navigation button -->
    <div class="mt-4">
      <button
        @click="navigateToExtract"
        class="px-4 py-2 bg-gray-500 text-white rounded-lg hover:bg-gray-600"
      >
        Go to Extract Page
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { useRouter } from "#app";

const imageSrc = ref(null);
const secretMessage = ref("");
const outputImage = ref(null);
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

function processImage() {
  if (!imageSrc.value || !secretMessage.value) return;

  const img = new Image();
  img.onload = () => {
    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;

    // Prepare the message with a null terminator
    const message = secretMessage.value + String.fromCharCode(0);
    const messageBits = message
      .split("")
      .map((c) => c.charCodeAt(0).toString(2).padStart(8, "0"))
      .join("");

    // Embed the message
    let messageIndex = 0;
    for (let i = 0; i < data.length; i += 4) {
      if (messageIndex < messageBits.length) {
        data[i] = (data[i] & ~1) | parseInt(messageBits[messageIndex]);
        messageIndex++;
      }
    }

    ctx.putImageData(imageData, 0, 0);
    outputImage.value = canvas.toDataURL("image/png");
  };
  img.src = imageSrc.value;
}

function navigateToExtract() {
  router.push("/extract");
}
</script>

<style scoped>
/* Add any additional styles if needed */
</style>
