<template>
  <div v-if="loggedIn" class="flex flex-col flex-1">
    <Header
        @logout="logout"
        @saveChanges="saveChanges"
        @resetChanges="resetChanges"
        :hasUnsavedChanges="hasUnsavedChanges"
    />
    <div class="flex flex-1 bg-gray-light">
      <Sidebar :currentPage="currentPage" @navigate="setCurrentPage"/>
      <div id="content" v-if="contentVisible" class="w-full">
        <ExampleTabs v-if="currentPage === 'example tabs'"/>
        <ExampleCards v-if="currentPage === 'example cards'"/>
        <ExampleTable v-if="currentPage === 'example table'"/>
        <ExampleEditableTable v-if="currentPage === 'example editable table'" v-model="store.examples.editableTable"/>
        <ExampleBars v-if="currentPage === 'example bars'" :values="store.examples.bars"/>
        <ExampleInputText v-if="currentPage === 'example input text'" v-model="store.examples.text"/>
        <ExampleInputNumber v-if="currentPage === 'example input numbers'" v-model="store.examples.numbers"/>
        <ExampleInputCheckbox v-if="currentPage === 'example input checkbox'" v-model="store.examples.checkbox"/>
        <ExampleInputSelect v-if="currentPage === 'example input select'" v-model="store.examples.selects"/>
        <ExampleSaveData v-if="currentPage === 'example save data'" v-model="store.contacts"/>
      </div>
    </div>

<!--    uncomment this to see data in store-->

<!--    <div class="border-1 border-b-0 border-black">-->
<!--      {{ store.examples.selects }}-->
<!--    </div>-->
<!--    <div class="border-1 border-black">-->
<!--      {{ initStore.examples.selects }}-->
<!--    </div>-->
  </div>
  <div v-else class="bg-gray-dark flex flex-1 justify-center items-center">
    <Card title="Please log in">
      <form @submit="login" class="flex flex-col items-end">
        <input v-model="credentials.login" placeholder="Login" name="login" class="input-text mb-2"/>
        <input v-model="credentials.password" placeholder="Password" type="password" name="password"
               class="input-text mb-2"/>
        <Button :callback="login">Log in</Button>
      </form>
    </Card>
  </div>
</template>

<script>
import Header from '@/components/Header'
import Sidebar from '@/components/Sidebar'
import {computed, reactive, ref} from 'vue'
import general from "@/api/general";
import auth from '@/api/auth'
import ExampleTabs from "@/components/pages/ExampleTabs";
import ExampleCards from "@/components/pages/ExampleCards";
import Card from "@/components/Card";
import Button from "@/components/Button";
import ExampleSaveData from "@/components/pages/ExampleSaveData";
import ExampleTable from "@/components/pages/ExampleTable";
import ExampleBars from "@/components/pages/ExampleBars";
import ExampleInputText from "@/components/pages/ExampleInputText";
import ExampleInputNumber from "@/components/pages/ExampleInputNumber";
import ExampleInputCheckbox from "@/components/pages/ExampleInputCheckbox";
import ExampleInputSelect from "@/components/pages/ExampleInputSelect";
import ExampleEditableTable from "@/components/pages/ExampleEditableTable";

export default {
  name: 'App',
  components: {
    ExampleEditableTable,
    ExampleInputSelect,
    ExampleInputCheckbox,
    ExampleInputNumber,
    ExampleInputText,
    ExampleBars,
    ExampleTable,
    ExampleSaveData,
    Card,
    ExampleCards,
    ExampleTabs,
    Header,
    Sidebar,
    Button,
  },
  setup() {
    // Navigation
    const defaultPage = 'example tabs'
    const currentPage = ref(defaultPage)

    function setCurrentPage(page) {
      currentPage.value = page
    }

    // Auth
    const loggedIn = ref(true)
    const credentials = reactive({})

    function login() {
      if (auth.login(credentials)) {
        loggedIn.value = true
        credentials.login = ''
        credentials.password = ''
      }
    }

    function logout() {
      if (auth.logout()) {
        loggedIn.value = false
      }
    }

    // Examples
    // Bars
    function updateBarsValues() {
      function getNumber() {
        return Math.round(Math.random() * 101);
      }
      const newValues = {
        first: getNumber(),
        second: getNumber(),
        third: getNumber(),
      }
      store.value.examples.bars = newValues
      initStore.value.examples.bars = newValues
    }
    setInterval(() => {
      updateBarsValues()
    }, 5000)

    // Store setup
    const store = ref({})
    const initStore = ref({})
    const contentVisible = ref(true)

    const hasUnsavedChanges = computed({
      get: () => !(JSON.stringify(store.value) === JSON.stringify(initStore.value))
    })

    function saveChanges() {
      if (general.saveChanges(store.value)) {
        initStore.value = JSON.parse(JSON.stringify(store.value)) // storing updated data without reference
      }
    }

    function resetChanges() {
      store.value = JSON.parse(JSON.stringify(initStore.value))

      // this forces Vue to re-render whole content div to make sure that v-models updated
      contentVisible.value = false
      setTimeout(() => contentVisible.value = true, 1)
    }

    general.init()
        .then((data) => {
          store.value = data
          initStore.value = JSON.parse(JSON.stringify(data)) // storing initial data without reference
        })
        .catch((reason) => console.log(reason))

    return {
      // Navigation
      currentPage,
      setCurrentPage,
      // Auth
      loggedIn,
      credentials,
      login,
      logout,
      // Store
      store,
      initStore,
      hasUnsavedChanges,
      saveChanges,
      resetChanges,
      contentVisible,
    }
  }
}
</script>
