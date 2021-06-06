<template>
  <q-page class="q-mt-lg">
    <div class="container">
      <div class="row justify-center">
        <q-card class="col-8">
          <!-- Tabs Section -->
          <q-tabs
            v-model="currentTab"
            dense
            class="text-grey"
            active-color="primary"
            indicator-color="primary"
            align="left"
            narrow-indicator
          >
            <q-tab
              v-for="tab in tabs"
              :key="tab.id"
              :name="tab.name"
              :label="tab.name"
            />
          </q-tabs>

          <q-separator />

          <q-tab-panels v-model="currentTab" animated>
            <q-tab-panel
              v-for="tab in tabs"
              :key="tab.id"
              :name="tab.name"
              style="height: 40rem"
            >
              <div class="text-h6">{{ tab.name }}</div>
              <q-list bordered>
                <q-item
                  clickable
                  v-ripple
                  v-for="card in tab.data"
                  :key="card.id"
                >
                  <q-item-section avatar>
                    <q-avatar color="teal" text-color="white" icon="movie" />
                  </q-item-section>
                  <q-item-section>{{ card.title }}</q-item-section>
                </q-item>

                <!-- An extra item to add new one's -->
                <q-item clickable v-ripple @click="cardPrompt = true">
                  <q-item-section>Add New Link</q-item-section>
                  <q-item-section avatar>
                    <q-icon color="primary" name="add" />
                  </q-item-section>
                </q-item>
              </q-list>
            </q-tab-panel>
          </q-tab-panels>

          <q-separator />

          <!-- Action Button Section. -->
          <div class="row">
            <div class="col inline-block q-pa-md q-gutter-sm">
              <q-btn round color="primary" icon="add" @click="prompt = true" />
              <q-btn round color="red" icon="delete" @click="confirm = true" />
              <q-btn round color="primary" icon="update" @click="openUpdatePrompt()" />

              <q-dialog v-model="prompt" persistent>
                <q-card style="min-width: 350px">
                  <q-card-section>
                    <div class="text-h6">Name a category.</div>
                  </q-card-section>

                  <q-card-section class="q-pt-none">
                    <q-input
                      outlined
                      autofocus
                      v-model="newCategoryName"
                      label="Category Name"
                    />
                  </q-card-section>

                  <q-card-actions align="right" class="text-primary">
                    <q-btn flat label="Cancel" v-close-popup />
                    <q-btn
                      flat
                      label="Add Category"
                      @click="addNewCategory()"
                    />
                  </q-card-actions>
                </q-card>
              </q-dialog>
              <q-dialog v-model="confirm" persistent>
                <q-card>
                  <q-card-section class="row items-center">
                    <q-avatar
                      icon="warning"
                      color="yellow"
                      text-color="white"
                    />
                    <span class="q-ml-sm"
                      >Are you sure you wanna delete <strong>{{this.currentTab}}</strong> Category ?</span
                    >
                  </q-card-section>

                  <q-card-actions align="right">
                    <q-btn flat label="Cancel" color="primary" v-close-popup />
                    <q-btn
                      flat
                      label="Delete"
                      color="primary"
                      v-close-popup
                      @click="deleteCategory()"
                    />
                  </q-card-actions>
                </q-card>
              </q-dialog>
              <q-dialog v-model="updateCategoryPrompt" persistent>
                <q-card style="min-width: 350px">
                  <q-card-section>
                    <div class="text-h6">Update the category.</div>
                  </q-card-section>

                  <q-card-section class="q-pt-none">
                    <q-input
                      outlined
                      autofocus
                      v-model="updateCategoryTitle"
                      label="Category name"
                      hint="Update the category."
                    />
                  </q-card-section>

                  <q-card-actions align="right" class="text-primary">
                    <q-btn flat label="Cancel" v-close-popup />
                    <q-btn flat label="Add Link" @click="updateCategory()" />
                  </q-card-actions>
                </q-card>
              </q-dialog>
              <q-dialog v-model="cardPrompt" persistent>
                <q-card style="min-width: 350px">
                  <q-card-section>
                    <div class="text-h6">Add a new Link.</div>
                  </q-card-section>

                  <q-card-section class="q-pt-none">
                    <q-input
                      outlined
                      autofocus
                      v-model="newCardTitle"
                      label="Video Title"
                      hint="Add a custom title."
                    />
                  </q-card-section>
                  <q-card-section class="q-pt-none">
                    <q-input
                      outlined
                      v-model="newCardLink"
                      label="Video Source"
                      hint="Source or link."
                    />
                  </q-card-section>

                  <q-card-actions align="right" class="text-primary">
                    <q-btn flat label="Cancel" v-close-popup />
                    <q-btn flat label="Add Link" @click="addNewCard()" />
                  </q-card-actions>
                </q-card>
              </q-dialog>
            </div>
          </div>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "PageIndex",
  mounted() {
    this.$api.get("api/category").then((response) => {
      const data = response.data;
      data.forEach((item) => {
        this.tabs.push({
          id: item.id,
          name: item.name,
          data: [],
        });
      });
      this.currentTab = data[0].name
    });
  },
  watch: {
    currentTab(newValue, oldValue)  {
      console.log({newValue, oldValue});
      if (newValue) {
        let selectedTab = this.tabs.find((tab) => tab.name == newValue);
        this.$api.get(`/api/category/${selectedTab.id}/links`)
          .then((response) => {
            selectedTab.data = []
            response.data.forEach((link) => {
                selectedTab.data.push({
                id: link.id,
                title: link.description,
                link: link.link
              })
            })
          })
      }
    }
  },
  methods: {
    handleTabChange(value) {
      console.log(value);
    },
    addNewCategory() {
      if (this.newCategoryName) {
        this.$api
          .post("/api/category", {
            name: this.newCategoryName,
          })
          .then((response) => {
            this.tabs.push({
              id: response.data.id,
              name: response.data.name,
              data: [],
            });
            console.log(response);
            this.newCategoryName = "";
            this.prompt = false;
          })
          .catch((error) => {
            console.log("Error.");
          });
      }
    },
    deleteCategory() {
      if (this.currentTab) {
        let selectedTab = this.tabs.find((tab) => tab.name == this.currentTab);
        this.$api.delete(`api/category/${selectedTab.id}`)
          .then((response) => {
            this.tabs = this.tabs.filter((tab) => tab != selectedTab)
            console.log(response)
          })
          .catch((error) => {
            console.log(error);
          })
      }
    },
    updateCategory() {
      if (this.currentTab && this.updateCategoryTitle) {
        let selectedTab = this.tabs.find((tab) => tab.name == this.currentTab);
        this.$api.put(`api/category/${selectedTab.id}`, {
          name: this.updateCategoryTitle
        })
          .then((response) => {
            selectedTab.name = response.data.name
            this.updateCategoryPrompt = false
            console.log(response)
          })
          .catch((error) => {
            console.log(error);
          })
      }
    },
    addNewCard() {
      if (this.newCardTitle && this.newCardLink) {
        let tab = this.tabs.find((tab) => tab.name == this.currentTab);
        console.log("Current Tab Name: ", tab.name);
        tab.data.push({
          id: tab.data.length + 1,
          title: this.newCardTitle,
          link: this.newCardLink,
        });
        this.currentTab = data[0].name;
        this.newCardLink = "";
        this.newCardTitle = "";
        this.cardPrompt = false;
      }
    },
    openUpdatePrompt() {
      this.updateCategoryPrompt = true
      this.updateCategoryTitle = this.currentTab
    }
  },
  computed: {
    toUpperLowerCase(str) {
      str[0] = str[0].toUpperCase();
      return str;
    },
  },
  data() {
    return {
      updateCategoryTitle: '',
      updateCategoryPrompt: false,
      confirm: false,
      currentTab: '',
      prompt: false,
      cardPrompt: false,
      newCategoryName: "",
      newCardTitle: "",
      newCardLink: "",
      tabs: [],
    };
  },
});
</script>

<style lang="scss">
.inline-block {
  width: max-content;
}
</style>
