<template>
  <div>
    <select
      v-if="sections.length"
      v-model="model.selectedSection"
      class="uk-width-1-1"
    >
      <option disabled value="">Please select one</option>
      <option 
        v-for="item in sections" 
        :key="item._uid" :value="item._uid"
      >
        {{ item.nickName || item._uid }}
      </option>
    </select>
  </div>
</template>

<script>
export default {
  name: 'PageSectionPlugin',
  mixins: [window.Storyblok.plugin],
  data() {
    return {
      sections: []
    }
  },
  methods: {
    initWith() {
      return {
        plugin: 'page-section',
        selectedSection: '',
      }
    },
    pluginCreated() {
      this.crawlStory()

      setInterval(() => {
        this.$emit('get-context');
      }, 100);
    },
    crawlRecursive (item) {
      let sections = []

      if (item && typeof item === 'object') {
        if (item.component && item.component === 'Section') {
          sections.push(item)
        }

        for (const key in item) {
          if (Object.hasOwnProperty.call(item, key) && item[key]) {
            const prop = item[key]

            if (typeof prop === 'object') {
              if (Array.isArray(prop)) {
                prop.forEach(block => {
                  sections = sections.concat(this.crawlRecursive(block))
                })
              } else {
                sections = sections.concat(this.crawlRecursive(prop))
              }
            } 
          }
        }
      }
      
      return sections
    },
    crawlStory () {
      this.sections = this.crawlRecursive(this.storyItem.content)
    },
  },
  watch: {
    model: {
      handler(value) {
        this.$emit('changed-model', value)
      },
      deep: true,
    },
    storyItem: function(old, newVal) {
      if (window.JSON.stringify(old) != window.JSON.stringify(newVal)) {
        this.crawlStory()
      }
    }
  }
}
</script>
