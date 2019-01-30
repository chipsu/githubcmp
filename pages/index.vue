<template>
  <div class="container mx-auto">
    <form @submit.prevent="submit">
      <input ref="a" type="text" name="a" placeholder="Repository A" value="nuxt/nuxt.js">
      <input ref="b" type="text" name="b" placeholder="Repository A" value="zeit/next.js">
      <button>Compare</button>
      <div v-if="repos.length">
        <div class="flex flex-wrap">
          <div class="p-1 w-1/2"> </div>
          <div v-for="repo in sorted" :key="repo.id" class="p-1 w-1/4">
            {{ repo.full_name }}
          </div>
        </div>
        <div v-for="(field, key) in fields" :key="key" class="flex flex-wrap">
          <div class="p-1 w-1/2">{{ field }}</div>
          <div v-for="repo in repos" :key="repo.id" class="p-1 w-1/4" :class="fieldClass(key, repo)">
            {{ repo[key] }}
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      repos: [],
      fields: {
        issue_score_round: 'Issue score',
        open_issues_count: 'Open issues',
        total_issues: 'Total issues',
        stargazers_count: 'Stars',
        watchers_count: 'Watchers',
        forks: 'Forks'
      },
      modifiers: {
        issue_score_round: 1,
        open_issues_count: -1,
        total_issues: -1,
        stargazers_count: 1,
        watchers_count: 1,
        forks: 1
      },
    }
  },
  methods: {
    submit() {
      this.repos = []
      this.repo(this.$refs.a.value)
      this.repo(this.$refs.b.value)
      return false
    },
    repo(name) {
      const url = 'https://api.github.com/'
      axios.all([
        axios.get(`${url}repos/${name}`),
        axios.get(`${url}search/issues?q=repo:${name}+type:issue`)
      ]).then(axios.spread((info, issues) => {
        info.data.total_issues = issues.data.total_count
        info.data.issue_score = (info.data.total_issues - info.data.open_issues_count) / info.data.total_issues * 100
        info.data.issue_score_round = Math.ceil(info.data.issue_score)
        this.repos.push(info.data)
      }))
    },
    fieldClass(key, repo) {
      const mod = this.modifiers[key]
      const max = Math.max.apply(Math, this.repos.map(item => {
        return item[key] * mod
      }))
      return repo[key] * mod < max ? 'bg-red' : 'bg-green'
    }
  },
  computed: {
    sorted() {
      return this.repos.slice(0).sort((a, b) => {
        return a.issue_score - b.issue_score
      })
    }
  }
}
</script>
