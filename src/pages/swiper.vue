<script lang="ts" setup>
import type { CSSProperties } from 'vue'

interface ItemType {
  content: any
  styles: CSSProperties
}
const array = ref<ItemType[]>([
  {
    content: 0,
    styles: {
      opacity: 1,
      background: randomColor(),
    },
  },
  {
    content: 1,
    styles: {
      opacity: 1,
      background: randomColor(),
    },
  },
  {
    content: 2,
    styles: {
      opacity: 1,
      background: randomColor(),
    },
  },
  {
    content: 3,
    styles: {
      opacity: 1,
      background: randomColor(),
    },
  },
  {
    content: 4,
    styles: {
      opacity: 1,
      background: randomColor(),
    },
  },
])

// const current = ref(0)

function randomColor(): string {
  const letters = '0123456789ABCDEF'
  let color = '#'
  for (let i = 0; i < 6; i++)
    color += letters[Math.floor(Math.random() * 16)]

  return color
}

function onScroll(event: UIEvent) {
  nextTick(() => {
    const scrollTop = (event.target as HTMLDivElement).scrollTop
    const height = 300

    const constant = scrollTop / height

    const cardinal = Math.floor(constant) >= array.value.length ? array.value.length - 1 : Math.floor(constant)
    const cardinalNext = cardinal + 1

    const opacity = cardinalNext - constant > 0.5 ? cardinalNext - constant : 0.5

    const scrollWidthScale = constant - cardinal > 1 ? 0 : constant - cardinal
    const width = `calc(100% - ${(array.value.length - cardinal) * 40 * (scrollWidthScale)}px - 40px)`

    if (constant === 0)
      array.value[cardinal].styles.opacity = opacity
    else
      array.value[cardinal - 1 >= 0 ? cardinal - 1 : 0].styles.opacity = opacity
    array.value[cardinal].styles.width = width
  })
}
</script>

<template>
  <div relative box-border h-380px w-screen overflow-scroll bg-red @scroll="onScroll">
    <div
      v-for="(item, i) in array" :key="item.content"
      sticky
      m-auto
      m-t-20
      h-300px
      w-full
      text-center
      transition-transform-300
      :style="{ ...item.styles, top: `${i * 20}px` }"
    >
      {{ item.content }}
    </div>
    <div h-full />
  </div>
</template>

<style></style>
