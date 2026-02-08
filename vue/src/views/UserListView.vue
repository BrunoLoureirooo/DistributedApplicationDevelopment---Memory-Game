<template>
    <div class="min-h-screen p-4">
      <h1 class="text-2xl font-bold mb-4">Users</h1>

        <div class="w-full justify-between flex h-auto">
          <input 
            v-model="search"
            @input="fetchUsers"
            placeholder="Search users..."
            class="p-2 border rounded-md w-full"
          />
            <router-link to="/register" class="w-36 p-3 bg-blue-700 hover:bg-blue-600 text-white rounded-md transition duration-200  mx-2 text-center">Criate User</router-link>
        </div>

      
      <div class="flex flex-wrap gap-4 mb-4">
      </div>
  
      <!-- Lista de Jogos -->
      <div v-if="loading" class="text-gray-500">Loading...</div>
      <div v-if="!loading && users.length === 0" class="text-gray-500">
        No users found.
      </div>
      <ul v-else>
        <li v-for="user in users" :key="user.id" class="mb-2 border py-2">
            <div class="grid grid-cols-2 gap-x-2 box-content">
    <div class="flex box-content">
        <div class="text-center grid grid-cols-1 gap-y-10 ml-5 mr-10">
            <h2><strong>Profile Picture</strong> </h2>
            <img
                :src="getImage(user.photo_filename)"
                alt="profile picture"
                class="w-20 h-20 rounded-full mx-auto"
            />
        </div>

        <div>
            <br>
            <p><strong>User ID:</strong> {{ user.id }}</p>
            <p><strong>Name:</strong> {{ user.name }}</p>
            <p><strong>Nickname:</strong> {{ user.nickname }}</p>
            <p><strong>Email:</strong> {{ user.email }}</p>
            <p v-if="user.type === 'A'"><strong>User type:</strong> Administrator</p>
            <p v-else><strong>User type:</strong> Player</p>
        </div>
    </div>

    <div class="flex flex-col items-end mt-2 mr-5 box-content">
        <div class="mb-4 flex-grow">
            <button 
                v-if="user.type !== 'A'"
                @click="blockUser(user.id)"
                :class="user.blocked==0?'py-4 w-36 bg-red-500 text-white font-medium rounded-md shadow-sm hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-300':
                'py-4 w-36 bg-green-600 text-white font-medium rounded-md shadow-sm hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 transition duration-300'"
            >
                {{ user.blocked==0?"Block":"Unblock" }}
            </button>
        </div>
        <div class="flex-grow">
            <button
                @click="winpopUp(user.id)"
                class="py-4 w-36 bg-red-500 text-white font-medium rounded-md shadow-sm hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition duration-300"
            >
                Delete
            </button>
        </div>
    </div>
</div>

            
          
          
        </li>
      </ul>
  
      <!-- Paginação -->
      <div>
        <span class="flex items-center justify-center my-2">
          Page {{ page }} of {{ totalPages }}
        </span>
      </div>
      <div class="pagination flex w-full">
        <button class="w-1/2 py-3 bg-blue-700 hover:bg-blue-600 text-white rounded-md transition duration-200 my-2 mx-2" @click="loadPage(page - 1)" :disabled="page === 1">Previous</button>
        <button class="w-1/2 py-3 bg-blue-700 hover:bg-blue-600 text-white rounded-md transition duration-200 my-2 mx-2" @click="loadPage(page + 1)" :disabled="page === totalPages">Next</button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, inject} from "vue";
  import { useAuthStore } from "@/stores/auth";
  import { useRouter } from 'vue-router';

  import axios from "axios";
  
  // Estado
  const authStore = useAuthStore();
  const users = ref([]);
  const boards = ref([]);
  const search = ref("");

  
  const alertDialog = inject('alertDialog'); // Inject alert dialog
  const loading = ref(false);
  const page = ref(1);
  const totalPages = ref(0);
  const apiDomain = import.meta.env.VITE_API_DOMAIN;

  // Função para buscar jogos
  const fetchUsers = async () => {
  loading.value = true;
  try {
    const { data } = await axios.get("/users", {
      params: {
        page: page.value,
        search: search.value,
      },
    });
    users.value = data.data;
    totalPages.value = data.meta.last_page;
  } catch (error) {
    console.error("Error fetching users:", error);
  } finally {
    loading.value = false;
  }
};

  
  // Função para buscar boards
  const fetchBoards = async () => {
    try {
      const { data } = await axios.get("/boards");
      boards.value = data.data;
    } catch (error) {
      console.error("Error fetching boards:", error);
    }
  };
  
  // Navegar para outra página
  const loadPage = async (newPage) => {
    if (newPage >= 1 && newPage <= totalPages.value) {
      page.value = newPage;
      await fetchUsers();
    }
  };


  const winpopUp = (id) => {
    console.log(alertDialog.value)
    alertDialog.value.open(
      ref(null),
      () => deleteUser(id),
      
      'Confirmation Needed',
      'Cancel',
      'Delete',
      `Are you sure? This user will be deleted`
    );
  };

  const getImage = (photo_filename) => {
    if (!photo_filename) {
      return '../assets/avatar-none.png';
    }
    return `http://${apiDomain}/storage/photos/${photo_filename}`;
  };

  const deleteUser = async (id) => {
    try {
      await authStore.deleteAccount(id);
      await fetchUsers();
    } catch (err) {
      console.error('Error deleting account:', err);
    }
  };

  const blockUser = async (id) => {
    try {
      await authStore.blockUser(id);
      await fetchUsers();
    } catch (err) {
      console.error('Error blocking account:', err);
    }
  }
  
  // Busca inicial
  onMounted(() => {
    fetchUsers();
    fetchBoards();
  });
  </script>
  