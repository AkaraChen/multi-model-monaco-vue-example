<script setup lang="ts">
import { VueMonacoEditor, useMonaco } from '@guolao/vue-monaco-editor'
import { onUnmounted, ref, shallowRef } from 'vue'
import type * as monaco from 'monaco-editor'

interface File {
  path: string
}

const { monacoRef } = useMonaco()

const files = ref<File[]>([])

const editorRef = shallowRef<monaco.editor.IEditor>()

const onload = (e: monaco.editor.IEditor) => {
  const monaco = monacoRef.value
  if (!monaco) return
  files.value = ['post.ts', 'posts.ts', 'upload.ts'].map((path) => {
    const content = `console.log('this is ${path}')`
    const uri = monaco.Uri.parse(path)
    const model = monaco.editor.createModel(
      content,
      path.endsWith('.ts') ? 'typescript' : 'javascript',
      uri
    )
    modelMap.set(path, model)
    model.onDidChangeContent(() => {
      modelMap.set(path, editorRef.value!.getModel() as monaco.editor.ITextModel)
    })
    return {
      path: path
    } as File
  })
  editorRef.value = e
  switchTab(files.value.at(0)!)
}

const modelMap = new Map<string, monaco.editor.ITextModel>()
const viewStateMap = new Map<string, monaco.editor.IEditorViewState>()

const activeFile = shallowRef<File>()
const switchTab = (to: File) => {
  const previous = activeFile.value
  const viewState = editorRef.value?.saveViewState()
  if (viewState && previous?.path) {
    viewStateMap.set(previous.path, viewState)
  }
  activeFile.value = to
  const activeModel = modelMap.get(to.path)
  if (activeModel) {
    editorRef.value?.setModel(activeModel)
  }
  const activeViewState = viewStateMap.get(to.path)
  if (activeViewState) {
    editorRef.value?.restoreViewState(activeViewState)
  }
}

onUnmounted(() => {
  editorRef.value?.dispose()
  for (const model of modelMap.values()) {
    model.dispose()
  }
})
</script>

<template>
  <div style="height: 100vh; width: 100vw; display: flex; flex-direction: column">
    <div style="height: 32px; background: skyblue">
      <ul style="display: flex">
        <li
          v-for="file in files"
          :key="file.path"
          @click="switchTab(file)"
          :style="{
            color: activeFile?.path === file.path ? 'red' : 'white',
            listStyle: 'none',
            marginInline: '4px'
          }"
        >
          {{ file.path }}
        </li>
      </ul>
    </div>
    <vue-monaco-editor style="height: 100%" @mount="onload" />
  </div>
</template>
