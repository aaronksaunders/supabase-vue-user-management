<template>
  <img
    :src="avatarUrl"
    alt="Avatar"
    class="avatar image"
    style="height: 150px, width: 150px"
  />
  <div style="width: 150px">
    <input
      style="visibility: hidden; position: absolute"
      type="file"
      id="single"
      accept="image/*"
      @change="uploadAvatar"
      :disabled="uploading"
    />
    <label class="button primary block" htmlFor="single">
      <span>{{ uploading ? "UpLoading..." : "Upload" }}</span>
    </label>
  </div>
</template>

<script>
import { ref, watch } from "@vue/runtime-core";
import { supabase } from "./supabaseClient";
import missingImage from "@/assets/no_image_available.jpeg";

export default {
  name: "Avatar",
  props: {
    url: String
  },
  emits: ["onUpload"],
  setup(props, ctx) {
    const avatarUrl = ref(null);
    const uploading = ref(false);

    watch(
      () => props?.url,
      (cur) => {
        downloadImage(cur);
      }
    );

    /**
     *
     */
    const downloadImage = async path => {
      console.log("download path", path);

      if (!path) {
        avatarUrl.value = missingImage;
        return;
      }

      const { data, error } = await supabase.storage
        .from("avatars")
        .download(path);
      if (error) throw error;
      avatarUrl.value = URL.createObjectURL(data);
    };

    async function uploadAvatar(event) {
      debugger;
      try {
        uploading.value = true;

        if (!event.target.files || event.target.files.length === 0) {
          throw new Error("You must select an image to upload.");
        }

        const file = event.target.files[0];
        const fileExt = file.name.split(".").pop();
        const fileName = `${Math.random()}.${fileExt}`;
        const filePath = `${fileName}`;

        let { error: uploadError } = await supabase.storage
          .from("avatars")
          .upload(filePath, file);

        if (uploadError) {
          throw uploadError;
        }

        ctx.emit("onUpload", filePath);
      } catch (error) {
        alert(error.message);
      } finally {
        uploading.value = false;
      }
    }

    return {
      avatarUrl,
      uploading,
      uploadAvatar
    };
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
