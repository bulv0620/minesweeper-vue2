<template>
    <div class="minesweeper" :style="{ width: panelWidth, height: panelHeight }">
        <div class="lattice"
            :class="{ opened: item.isOpened, flag: item.isMarked, bomb: item.isBomb, mine_red: item.mine_red }"
            v-for="item in lattice" :key="item.index" @click.left="latticeClick(item)"
            @click.right.prevent="setFlag(item)" @dblclick="autoReveal(item)">
            <span v-if="item.isOpened && item.minesAround > 0 && !item.isMine" :class="'n' + item.minesAround">
                {{ item.minesAround }}
            </span>
        </div>
    </div>
</template>

<script>
export default {
    props: ['status'],
    data() {
        return {
            MAX_MINES: 10,
            ROWS: 10,
            COLS: 10,
            lattice: [],
            minePos: [],
        }
    },
    computed: {
        // 计算游戏面板的宽度和高度
        panelWidth() {
            return (this.COLS * 35) + 'px';
        },
        panelHeight() {
            return (this.ROWS * 35) + 'px';
        },
        // 计算已经扫过的格子数量
        openedLatticeCount() {
            return this.lattice.filter(item => item.isOpened).length;
        },
    },
    beforeMount() {
        this.initLattice();
        // console.log(this.lattice);
        // console.log(this.minePos)
    },
    watch: {
        // 监听以扫个数，如果扫完了，则游戏胜利
        openedLatticeCount() {
            if (this.openedLatticeCount === this.ROWS * this.COLS - this.MAX_MINES && this.status === 'ready') {
                this.$emit('update:status', 'win');
            }
        },
    },
    methods: {
        // 重置游戏
        reset() {
            this.minePos = [];
            this.lattice = [];
            this.initLattice();
            this.$emit('update:status', 'ready');
        },

        // 游戏结束
        gameover(lattice) {
            this.minePos.forEach(item => {
                this.lattice[item].isBomb = true;
            })
            this.$emit('update:status', 'over');
        },

        // 探测
        reveal(lattice) {
            if (lattice.isOpened || lattice.isMarked || this.status !== 'ready') {
                return;
            }
            if (lattice.isMine) {
                this.gameover(lattice);
                lattice.mine_red = true;
            }
            lattice.isOpened = true;
            if (lattice.minesAround === 0) {
                this.getAroundPosList(lattice).forEach(item => {
                    this.reveal(this.lattice[item]);
                })
            }
        },

        // 单击扫雷
        latticeClick(lattice) {
            if (lattice.isMarked || this.status !== 'ready') {
                return;
            }
            if (lattice.isOpened) {
                this.autoReveal(lattice)
            }
            this.reveal(lattice);
        },

        // 双击扩图
        autoReveal(lattice) {
            const aroundList = this.getAroundPosList(lattice);
            const flagNum = aroundList.filter(item => this.lattice[item].isMarked).length;
            if (flagNum >= lattice.minesAround) {
                aroundList.forEach(item => {
                    if (!this.lattice[item].isMarked) {
                        this.reveal(this.lattice[item])
                    }
                })
            }
        },

        // 右键插旗
        setFlag(lattice) {
            if (lattice.isOpened || this.status !== 'ready') {
                return;
            }
            lattice.isMarked = !lattice.isMarked;
        },


        getAroundPosList(lattice) {
            const pos = lattice.index;
            const row = Math.floor(pos / this.COLS);
            const col = pos % this.COLS;
            const aroundPosList = [];
            for (let i = row - 1; i <= row + 1; i++) {
                for (let j = col - 1; j <= col + 1; j++) {
                    if (i < 0 || i >= this.ROWS || j < 0 || j >= this.COLS) {
                        continue;
                    }
                    const aroundPos = i * this.COLS + j;
                    aroundPosList.push(aroundPos);
                }
            }
            return aroundPosList;
        },

        // 初始化
        initLattice() {
            const num = this.ROWS * this.COLS;
            for (let i = 0; i < num; i++) {
                this.lattice.push({
                    index: i,
                    minesAround: 0,
                    isMine: false,
                    isOpened: false,
                    isMarked: false,
                    isBomb: false
                })
            }
            this.generateMines();
            this.lattice.forEach(item => {
                this.countMinesAround(item)
            })
        },

        generateMines() {
            while (this.minePos.length < this.MAX_MINES) {
                const pos = Math.floor(Math.random() * this.ROWS * this.COLS);
                if (this.lattice[pos].isMine) {
                    continue;
                }
                this.minePos.push(pos);
                this.lattice[pos].isMine = true;
            }
        },

        countMinesAround(lattice) {
            this.getAroundPosList(lattice).forEach(pos => {
                if (this.lattice[pos].isMine) {
                    lattice.minesAround++;
                }
            })
        },

    }
}
</script>

<style>
.minesweeper {
    border: 10px solid darkslategray;
    display: flex;
    flex-wrap: wrap;
}

.minesweeper .lattice {
    width: 35px;
    height: 35px;
    background: url(../assets/closed.svg) no-repeat;
    background-size: cover;
    font-size: 24px;
    text-align: center;
    line-height: 35px;
    font-weight: bold;
}

.minesweeper .lattice.opened {
    background: url(../assets/opened.svg) no-repeat;
    background-size: cover;
}

.minesweeper .lattice.flag {
    background: url(../assets/flag.svg) no-repeat;
    background-size: cover;
}

.minesweeper .lattice.bomb {
    background: url(../assets/mine.svg) no-repeat;
    background-size: cover;
}

.minesweeper .lattice.mine_red {
    background: url(../assets/mine_red.svg) no-repeat;
    background-size: cover;
}

.minesweeper .lattice span {
    cursor: default;
}

.minesweeper .lattice span.n1 {
    color: blue;
}

.minesweeper .lattice span.n2 {
    color: green;
}

.minesweeper .lattice span.n3 {
    color: red;
}

.minesweeper .lattice span.n4 {
    color: darkblue;
}

.minesweeper .lattice span.n5 {
    color: purple;
}

.minesweeper .lattice span.n6 {
    color: seagreen;
}

.minesweeper .lattice span.n7 {
    color: yellow;
}

.minesweeper .lattice span.n8 {
    color: brown;
}
</style>