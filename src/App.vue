<script setup lang="ts">
import { onMounted, reactive, watch } from 'vue';

const dict = reactive({
  working: false,
  magnet_link: "",
  trackers_text: "",
  tracker_links_list: [] as string[]
})

const functions = reactive({
  is_it_magnet_link: (text: string) => {
    if (text.trim().startsWith("magnet:?xt=")) {
      return true
    } else {
      return false
    }
  }, 
  parse_magnet_link: (text: string) => {
    text = text.trim()
    let tracker_regex = /&tr=(?<link>[^&]+)/g
    let matches = text.matchAll(tracker_regex)

    let tracker_list = [] as string[]
    for (const match of matches) {
      let one = match?.groups?.link
      if (one) {
        one = decodeURIComponent(one)
        tracker_list.push(one)
      }
    }

    for (const link of tracker_list) {
      if (!dict.tracker_links_list.includes(link)) {
        dict.tracker_links_list.push(link)
      }
    }

    functions.save_trackers_into_storage()

    functions.update_tracker_text()

    functions.update_magnet_link()
  },
  update_tracker_text: () => {
    dict.trackers_text = dict.tracker_links_list.join("\n\n")
  },
  save_trackers_into_storage: () => {
    localStorage.setItem("tracker_list", JSON.stringify(dict.tracker_links_list))
  },
  load_trackers_from_storage: () => {
    let data = localStorage.getItem("tracker_list")
    if (data) {
      dict.tracker_links_list = JSON.parse(data)
    }
    functions.update_tracker_text()
  },
  parse_trackers_text: () => {
    let text = dict.trackers_text.trim()
    let lines = text.split('\n')
    lines = lines.filter((line) => line.trim() != "")

    for (const line of lines) {
      let link = decodeURIComponent(line)
      if (!dict.tracker_links_list.includes(link)) {
        dict.tracker_links_list.push(link)
      }
    }

    functions.save_trackers_into_storage()

    functions.update_magnet_link()
  },
  get_base_magnet_link: (text: string) => {
    let splitor = "&tr="
    let base_url = ""
    if (text.includes(splitor)) {
      base_url = text.split(splitor)[0]
    } else {
      base_url = text
    }
    return base_url
  },
  update_magnet_link: () => {
    let new_list = dict.tracker_links_list.map((link) => encodeURIComponent(link))
    let magnet_tail_text = "&tr=" + new_list.join("&tr=")
    dict.magnet_link = functions.get_base_magnet_link(dict.magnet_link) + magnet_tail_text
  }
})

onMounted(() => {
  functions.load_trackers_from_storage()

  watch(()=>dict.magnet_link, (new_value: any, old_value: any) => {
    if (dict.working == false) {
      dict.working = true
      functions.parse_magnet_link(new_value)
      dict.working = false
    }
  })

  watch(()=>dict.trackers_text, (new_value: any, old_value: any) => {
    if (dict.working == false) {
      dict.working = true
      functions.parse_trackers_text()
      dict.working = false
    }
  })
})
</script>

<template>
  <div class="container">
    <div class="box left">
      <p>Magnet Link:</p>
      <a-textarea
        v-model:value="dict.magnet_link"
        placeholder=""
        :auto-size="{ minRows: 5, maxRows: 10 }"
      />
    </div>
    <div class="box right">
      <p>Tracker Links:</p>
      <a-textarea
        v-model:value="dict.trackers_text"
        placeholder=""
        :auto-size="{ minRows: 15, maxRows: 25 }"
      />
    </div>
  </div>
</template>

<style scoped>
.container {
  width: 100vw;
  height: 100vh;

  display: flex;
  flex-direction: column;
  justify-content: flex-start;

  margin-top: 20px;

  padding: 16px;
}
.box {
  width: 100%;
  margin-bottom: 40px;
}
</style>