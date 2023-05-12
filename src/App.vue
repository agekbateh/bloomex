<template>
  <div class="wrapper">
    <div class="controls">
      <input v-model="title" placeholder="Issue title">
      <button @click="addIssue" class="btn add">Add issue</button>
    </div>
    <div class="issue-list" v-if="issues.length">
      <div class="issue" v-for="item in issues" :key="item.id">
        <h4 class="issue__header"><span>id:{{ item.id }}</span> {{ item.subject }}</h4>
        <button @click="removeIssue(item.id)" class="btn remove">Remove</button>
      </div>
    </div>
    <div class="issue-list" v-else>
      Good job, no more issues
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'App',
  data() {
    return {
      issues: [],
      loaded: false,
      title: ''
    }
  },
  mounted() {
    this.getData();
  },
  methods: {
    getData() {
      fetch('/issues.json')
          .then(response => response.json())
          .then(data => {
            this.issues = data.issues;
            this.loaded = true;
          })
    },
    addIssue() {
      const issue = {
        issue:
            {
              project_id: 1,
              subject: this.title ? this.title : 'Untitled',
              priority_id: 4
            }
      };
      fetch('/issues.json', {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "X-Redmine-API-Key": process.env.VUE_APP_API_KEY
        },
        body: JSON.stringify(issue),
      }).then(async resp => {
        const issueCreated = await resp.json().then(data => data.issue);
        this.title = "";
        if (resp.status === 201) { // create success
          this.issues = [...this.issues, {...issueCreated}];
        }
      }).catch(e => {
        console.warn(e)
      });
    },
    removeIssue(id) {
      fetch(`/issues/${id}.json`, {
        method: "DELETE",
        mode: "cors",
        credentials: "include",
        headers: {
          "Content-Type": "application/json",
          "X-Redmine-API-Key": process.env.VUE_APP_API_KEY
        },
      }).then(resp => {
        if (resp.status === 204) { // remove success
          this.issues = this.issues.filter(item => item.id != id);
        }
      }).catch(e => {
        console.warn(e)
      })
    }
  }
})
</script>

<style>

body {
  margin: 0;
  padding: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.wrapper {
  padding: 40px;
}

.controls {
  display: flex;
}

.issue-list {
  margin-top: 30px;
}

.issue {
  border: 1px solid lightgray;
  padding: 10px;
  border-radius: 4px;
}

.issue + .issue {
  margin-top: 20px;
}

.issue__header {
  margin-top: 0;
  margin-bottom: 5px;
  line-height: 1.35;
}

.remove {
  display: block;
}
</style>
