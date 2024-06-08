<script setup>
import { ref, onMounted, computed } from "vue";
import { useQuery } from "vue-query";
import { useRouter } from "vue-router";
import axios from "axios";
import { RouterLink } from "vue-router";

const gittoken = "cant push";
console.log(gittoken);

const fetchUserProfile = async () => {
  try {
    const response = await axios.get("https://api.github.com/users/Faidunni", {
      headers: {
        Authorization: `Bearer ${gittoken}`,
      },
    });
    return response.data;
  } catch (error) {
    console.error("Failed to fetch user profile:", error);
    throw new Error("Failed to fetch user profile");
  }
};

const fetchRepositories = async () => {
  try {
    const response = await axios.get(
      "https://api.github.com/users/Faidunni/repos",
      {
        headers: {
          Authorization: `Bearer ${gittoken}`,
        },
      }
    );
    console.log(response.data);
    return response.data;
  } catch (error) {
    console.error("Failed to fetch repositories:", error);
    throw new Error("Failed to fetch repositories");
  }
};

fetchRepositories();
const fetchRepositoryCount = async () => {
  try {
    const response = await axios.get("https://api.github.com/users/Faidunni", {
      headers: {
        Authorization: `Bearer ${gittoken}`,
      },
    });
    return response.data.public_repos;
  } catch (error) {
    console.error("Failed to fetch repository count:", error);
    throw new Error("Failed to fetch repository count");
  }
};

const userProfile = ref(null);
const searchQuery = ref("");
const repositoryCount = ref(0);
const router = useRouter();
const isOpen = ref(false);

const {
  data: repositories,
  isLoading,
  isError,
} = useQuery(["repositories"], fetchRepositories);

onMounted(async () => {
  try {
    const [userData, repoCountData] = await Promise.all([
      fetchUserProfile(),
      fetchRepositoryCount(),
    ]);
    userProfile.value = userData;
    repositoryCount.value = repoCountData;
  } catch (error) {
    console.error("Failed to fetch data:", error);
  }
});

const filteredRepositories = computed(() => {
  return (
    repositories.value?.filter((repo) =>
      repo.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    ) || []
  );
});
</script>

<template>
  <div class="header">
    <section>
      <h1 class="header-desktop">MY PORTFOLIO</h1>
      <section class="RepositoriesList">
        <h2>My Repositories</h2>
        <section class="profile-List">
          <div v-if="userProfile" class="profile">
            <h1>{{ userProfile.name }}</h1>
            <p>{{ userProfile.bio }}</p>
            <p>
              GitHub Profile:
              <a
                :href="userProfile.html_url"
                target="_blank"
                rel="noopener noreferrer"
              >
                {{ userProfile.html_url }}
              </a>
            </p>

            <input
              type="text"
              placeholder="Search repositories..."
              class="search-repo"
              v-model="searchQuery"
            />
            <ul>
              <li v-for="repo in paginatedRepositories" :key="repo.id">
                <span class="span-name">{{ repo.name.toUpperCase() }}</span>
                <button class="infoBTN">
                  <RouterLink class="link" :to="`/repository/${repo.id}`"
                    >DETAILS</RouterLink
                  >
                </button>
              </li>
            </ul>
          </div>
        </section>

        <ul>
          <li v-for="repo in paginatedRepositories" :key="repo.id">
            <span class="span-name">{{ repo.name.toUpperCase() }}</span>
            <button class="infoBTN">
              <RouterLink class="link" :to="`/repository/${repo.id}`"
                >DETAILS</RouterLink
              >
            </button>
          </li>
        </ul>
      </section>
    </section>
  </div>
</template>

<style></style>
