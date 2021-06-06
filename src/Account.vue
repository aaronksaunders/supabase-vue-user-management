
<template>
  <div class="form-widget">
    <h1 class="header">Supabase + Vue.js: Account</h1>
    <avatar :url="avatar_url" @onUpload="handleImageUpload" />
    <div>
      <label htmlFor="email">Email</label>
      <input id="email" type="text" :value="session.user.email" disabled />
    </div>
    <div>
      <label htmlFor="username">Name</label>
      <input id="username" type="text" v-model="username" />
    </div>
    <div>
      <label htmlFor="website">Website</label>
      <input id="website" type="website" v-model="website" />
    </div>

    <div>
      <button
        class="button block primary"
        @click="updateProfile({ username, website, avatar_url })"
        :disabled="loading"
      >
        <span>{{ loading ? "Loading..." : "Update" }}</span>
      </button>
    </div>

    <div>
      <button class="button block" @click="supabase.auth.signOut()">
        Sign Out
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import Avatar from "./Avatar";
import { supabase } from "./supabaseClient";

export default defineComponent({
  name: "Account",
  props: ["session"],
  components: {
    Avatar
  },
  setup(props) {
    const loading = ref(false);
    const username = ref("");
    const website = ref("");
    const avatar_url = ref("");

    /**
     *
     */
    const handleImageUpload = async path => {
      avatar_url.value = path;
      await updateProfile({ username, website, avatar_url: path });
    };

    const updateProfile = async ({ username, website, avatar_url }) => {
      try {
        debugger;
        loading.value = true;
        const user = supabase.auth.user();

        const updates = {
          id: user.id,
          username : username.value,
          website: website.value,
          avatar_url: (avatar_url.value || avatar_url),
          updated_at: new Date()
        };

        let { error } = await supabase.from("profiles").upsert(updates, {
          returning: "minimal" // Don't return the value after inserting
        });

        if (error) {
          throw error;
        }
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    };

    const getProfile = async session => {
      try {
        loading.value = true;
        const user = session.user;

        let { data, error, status } = await supabase
          .from("profiles")
          .select(`username, website, avatar_url`)
          .eq("id", user.id)
          .single();

        if (error && status !== 406) {
          throw error;
        }

        if (data) {
          username.value = data.username;
          website.value = data.website;
          avatar_url.value = data.avatar_url;
        }

        debugger;
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    };

    getProfile(props.session);

    return {
      loading,
      username,
      website,
      avatar_url,
      updateProfile,
      supabase,
      handleImageUpload
    };
  }
});
</script>

<style scoped>
</style>
