<script setup lang="ts">
interface BlockState{
  x: number
  y: number
  revealed?: boolean // 是否展示
  mine?: boolean
  flagged?: boolean
  adjacentMines: number
}

const WIDTH = 10
const HEIGHT = 10
const state = reactive(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH },
      (_, x): BlockState => ({
        x, y, adjacentMines: 0, revealed: false,
      }),
    ),
  ),
)

// direction 八个方向
const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-yellow-500',
]
const getSiblings = (block: BlockState) => {
  return directions.map(([dx, dy]) => {
    // 当前 x轴的位置
    const x2 = block.x + dx
    // 当前 y轴的位置
    const y2 = block.y + dy
    // 边界判断
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    // 如果踩到雷
    return state[y2][x2]
  }).filter(Boolean) as BlockState[]
}
// 更新雷附近的数字
const updateNumbers = () => {
  state.forEach((row, y) => {
    row.forEach((block, x) => {
      // console.log(x)
      // 如果是雷
      if (block.mine)
        return
      getSiblings(block).forEach((sibling) => {
        if (sibling.mine)
          block.adjacentMines++
      })
    })
  })
}
// 生成雷
const generateMines = (initial: BlockState) => {
  for (const row of state) {
    for (const block of row) {
      // 左右 上下两格不会有炸弹
      if (Math.abs(initial.x - block.x) < 1)
        continue
      if (Math.abs(initial.y - block.y) < 1)
        continue
      block.mine = Math.random() < 0.2
    }

    // console.log(block)
  }
  updateNumbers()
}
// 循环遍历展开 0
const expandZero = (block: BlockState) => {
  if (block.adjacentMines > 0)
    return
  getSiblings(block).forEach((sibling) => {
    if (sibling.revealed)
      return
    sibling.revealed = true
    expandZero(sibling)
  })
}
let mineGenerated = false

const dev = true

// 游戏开始
const onClick = (block: BlockState) => {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }

  block.revealed = true
  if (block.mine)
    alert('Boom!')
  expandZero(block)
}

// 样式
const getBlockClass = (block: BlockState) => {
  if (!block.revealed)
    return 'bg-gray/10'

  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}
// generateMines()

</script>

<template>
  <div>
    Minesweeper
    <div>
      <div
        v-for="row,y in state"
        :key="y"
        flex="~"
        items-center
        justify-center
      >
        <button
          v-for="block,x in row"
          :key="x"
          w-8 h-8
          border="1 gray-400/50"
          flex="~"
          items-center
          justify-center
          hover="bg-gray-400/10"
          :class="getBlockClass(block)"
          @click="onClick(block)"
        >
          <template v-if="block.revealed || dev">
            <div v-if="block.mine" i-mdi:mine />
            <div v-else>
              {{ block.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
