<script setup lang="ts">
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent } from '@tiptap/vue-3'

import VueComponent from './extension'

const editor = shallowRef()
const coms = ref([])
onMounted(() => {
  editor.value = new Editor({
    extensions: [
      StarterKit,
      VueComponent,
    ],
    content: `
        <p>
          This is still the text editor you’re used to, but enriched with node views.
        </p>
        <vue-component count="0" click="test"></vue-component>
        <p>
        </p>
        <vue-component count="100" click="test"></vue-component>
        <p>
          Did you see that? That’s a Vue component. We are really living in the future.
        </p>
      `,
    onCreate({ editor }) {
      console.log('editor: ', editor)
      coms.value = []
      editor.vueRenderers.forEach((vueRender) => {
        coms.value.push(vueRender.props.node.attrs)
      })
    },
    onUpdate({ editor }) {
      console.log('editor: ', editor)
      coms.value = []
      editor.vueRenderers.forEach((vueRender) => {
        coms.value.push(vueRender.props.node.attrs)
      })
    },
  })
})

function insert() {
  editor.value.commands.insertContent('<vue-component count="100" click="test"></vue-component>')
}

onBeforeUnmount(() => {
  editor.value.destroy()
})
</script>

<template>
  <button @click="insert">
    插入
  </button>
  <button v-for="comp in coms" :key="comp.count" border-1 h-50px @click="comp.count++">
    {{ comp }}
  </button>
  <EditorContent :editor="editor" />
</template>
