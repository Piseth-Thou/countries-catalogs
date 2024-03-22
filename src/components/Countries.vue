<template>
  <v-card>
    <v-card-title class="d-flex align-center pe-2">
      Countries Catalogs
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        density="compact"
        label="Search"
        prepend-inner-icon="mdi-magnify"
        variant="solo-filled"
        flat
        hide-details
        single-line
      ></v-text-field>
    </v-card-title>
    <v-divider></v-divider>
    <v-data-table
      style="min-width: 600px; width: 100%"
      :items-per-page="25"
      v-model:search="search"
      :items="filteredCountries"
      :headers="headers"
    >
      <template v-slot:header.flags>
        <div>Flags</div>
      </template>
      <template v-slot:header.name>
        <div @click="sortBy = 'name'">
          <v-icon icon="mdi-sort-alphabetical-variant" end></v-icon>
          <span class="ml-2">Country Name</span>
        </div>
      </template>
      <template v-slot:header.cca2>
        <div>CCA2</div>
      </template>
      <template v-slot:header.cca3>
        <div>CCA3</div>
      </template>
      <template v-slot:header.nativeName>
        <div>Native Country Name</div>
      </template>
      <template v-slot:header.altSpellings>
        <div>Alternative Country Name</div>
      </template>
      <template v-slot:header.idd>
        <div>Country Calling Code</div>
      </template>
      <template v-slot:header.action>
        <div>Action</div>
      </template>

      <template v-slot:item.flags="{ item }">
        <img :src="item.flags" alt="flags" width="35" />
      </template>
      <template v-slot:item.altSpellings="{ item }">
        <div v-for="data in item.altSpellings" :key="data">
          <div>- {{ data }}</div>
        </div>
      </template>

      <!-- Preview Dialog -->
      <template v-slot:item.action="{ item }">
        <v-dialog max-width="500">
          <template v-slot:activator="{ props: activatorProps }">
            <v-icon
              class="cursor-pointer"
              icon="mdi-eye-outline"
              end
              v-bind="activatorProps"
            ></v-icon>
          </template>

          <template v-slot:default="{ isActive }">
            <v-card title="Country Detail">
              <v-card-text>
                <div class="d-flex">
                  <img :src="item.flags" alt="flags" width="150" />
                  <div class="ml-4">
                    <li>Name : {{ item.name }}</li>
                    <li>CCA2 : {{ item.cca2 }}</li>
                    <li>CCA3 : {{ item.cca3 }}</li>
                    <li>Native Name : {{ item.nativeName }}</li>
                    <li>Country Code : {{ item.idd }}</li>
                  </div>
                </div>
                <div class="mt-4">Alternative Country Name</div>
                <div v-for="data in item.altSpellings" :key="data">
                  <li>{{ data }}</li>
                </div>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>

                <v-btn text="Close" @click="isActive.value = false"></v-btn>
              </v-card-actions>
            </v-card>
          </template>
        </v-dialog>
      </template>
    </v-data-table>
  </v-card>
</template>
<script setup>
import axios from "axios";
import { onMounted, ref, computed } from "vue";

const search = ref("");
const countries = ref([]);
const sortBy = ref("");

const headers = [
  {
    sortable: false,
    key: "flags",
  },
  {
    sortable: true,
    key: "name",
    value: "name",
  },
  {
    sortable: false,
    key: "cca2",
  },
  {
    sortable: false,
    key: "cca3",
  },
  {
    sortable: false,
    key: "nativeName",
  },
  {
    sortable: false,
    key: "altSpellings",
  },
  {
    sortable: false,
    key: "idd",
  },
  {
    sortable: false,
    key: "action",
  },
];

onMounted(() => {
  onGetCountries();
});

const onGetCountries = async () => {
  await axios
    .get("https://restcountries.com/v3.1/all")
    .then((res) => {
      const newData = res?.data?.map((obj) => ({
        flags: obj.flags.png,
        name: obj.name.official,
        cca2: obj.cca2,
        cca3: obj.cca3,
        nativeName: obj.name.common,
        altSpellings: obj.altSpellings,
        idd: obj.idd.root,
      }));
      countries.value = [...newData];
    })
    .catch((err) => {
      console.log("err", err);
    });
};

const filteredCountries = computed(() => {
  const keyword = search.value.toLowerCase().trim();
  if (!keyword) {
    return countries.value;
  }
  return countries.value.filter((country) =>
    country.name.toLowerCase().includes(keyword)
  );
});
</script>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
