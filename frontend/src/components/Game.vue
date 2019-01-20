<template>
  <div class="container">
    <div class="block-list">
      <div class="block"
        v-for="block in completedFlattenBlocks"
        @click="blockClickHandler(block)"
        :key="block"
        :style="getBlockStyle(block)">
        <div class="block-inner" :class="{'is-empty': !block}">
          <div class="block-text">{{block}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Game',
  data() {
    return {
      inPlay: false,
      blocks: [],
    };
  },
  watch: {
    isCompleted(newVal) {
      if (newVal && this.inPlay) {
        alert('completed!');
        this.inPlay = false;
      }
    },
  },
  computed: {
    completedFlattenBlocks() {
      return Array.from({length: 16}).map((_, i) => (i + 1) % 16);
    },
    completedBlocks() {
      return this.completedFlattenBlocks.reduce((acc, cur, i) => {
        const bi = ~~(i / 4);
        acc[bi] = acc[bi] || [];
        acc[bi].push(cur);
        return acc;
      }, []);
    },
    flattenBlocks() {
      return this.blocks.reduce((acc, cur) => acc.concat(cur), []);
    },
    isCompleted() {
      return !this.completedFlattenBlocks.filter((n, i) => n !== this.flattenBlocks[i]).length;
    },
  },
  mounted() {
    this.blocks = this.completedBlocks;
    this.shuffle();
    this.inPlay = true;
  },
  methods: {
    blockClickHandler(block) {
      this.moveBlock(block);
    },
    arrayDeepCopy(array) {
      return Array.isArray(array) ? array.map(this.arrayDeepCopy) : array;
    },
    shuffle() {
      let i = 50;
      while (i) {
        const targetBlock = ~~(Math.random() * 15) + 1;
        i -= this.moveBlock(targetBlock) ? 1 : 0;
      }
    },
    xor(a, b) {
      return (!a && b) || (a && !b);
    },
    moveBlock(block) {
      const emptyPos = this.getBlockPos(0);
      const blockPos = this.getBlockPos(block);
      const diffX = emptyPos.x - blockPos.x;
      const diffY = emptyPos.y - blockPos.y;
      const isMovable = this.xor(diffX, diffY);

      if (!isMovable) return false;

      const incrementX = this.getIncrementNum(diffX);
      const incrementY = this.getIncrementNum(diffY);
      const loopCount = Math.abs(diffX + diffY);

      let newBlocks = this.arrayDeepCopy(this.blocks);
      let prevBlock = block;
      for (let i = 1; i <= loopCount; i++) {
        const curX = incrementX * i + blockPos.x;
        const curY = incrementY * i + blockPos.y;
        newBlocks[curY][curX] = prevBlock;
        prevBlock = this.blocks[curY][curX];
      }
      newBlocks[blockPos.y][blockPos.x] = 0;
      this.blocks = newBlocks;
      return newBlocks;
    },
    getIncrementNum(n) {
      return n === 0 ? 0 : (n > 0 ? 1 : -1);
    },
    getBlockPos(searchBlock) {
      let r = null;
      this.blocks.forEach((row, y) => row.forEach((block, x) => {
        if (block === searchBlock) r = {y, x};
      }));
      return r;
    },
    getBlockStyle(block) {
      const pos = this.getBlockPos(block);
      return pos && {
        left: `${25 * pos.x}%`,
        top: `${25 * pos.y}%`,
      }
    },
  },
}
</script>

<style scoped>
.block-list {
  padding: 2px;
  box-sizing: border-box;
  background: #efefef;
  display: block;
  position: relative;
}
.block-list:before {
  content: '';
  display: block;
  padding-top: 100%;
}

.block {
  width: 25%;
  height: 25%;
  position: absolute;
  transition: all 0.1s ease;
  box-sizing: border-box;
  padding: 2px;
}

.block-inner {
  background: #fff;
  box-sizing: border-box;
  position: relative;
  line-height: 1;
  height: 100%;
  width: 100%;
}

.block-text {
  position: absolute;
  top: 50%;
  left: 0;
  width: 100%;
  margin-top: -0.5em;
  text-align: center;
  font-size: 28px;
  font-weight: bold;
}

.block-inner.is-empty {
  opacity: 0;
}

.container {
  margin: 0 auto;
  max-width: 680px;
}
</style>
