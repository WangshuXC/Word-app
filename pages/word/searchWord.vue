<template>
	<view class="blank"></view>
	<view class="search-box">
		<input type="text" class='search-bar' v-model="word" placeholder="输入单词, 开始查询" @confirm="search(this.word)" />
		<view class="back-btn" @click="back()">
			取消
		</view>
	</view>
	<view class="sort-bar">
		<view class="sort-bar-btn" :class="{ 'sort-bar-btn-selected': sortItemSelected[index] }"
			@click="selected(index)" v-for="(item,index) in sortItem" :key="index">
			{{item}}
		</view>
	</view>
	<view class="word-box">
		<view class="word-item" v-for="(item,index) in wordList" :key='index' @click="goToWordDetail(item.wordId)">
			<p>{{item.word}}</p>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				word: '',
				wordList: [],
				sortItem: ['单词', '词义', '例句', '备注'],
				sortItemSelected: [true, true, true, true],
			}
		},
		onLoad() {
			this.openDB();
		},
		// watch: {
		// 	word(newWord) {
		// 		if (newWord !== '') {
		// 			this.search(newWord);
		// 		} else {
		// 			this.wordList = [];
		// 		}
		// 	}
		// },
		methods: {
			openDB() {
				plus.sqlite.openDatabase({
					name: 'first',
					path: '_doc/data.db',
				});
			},
			back() {
				uni.navigateBack();
			},
			search(inputWord) {
				console.log(this.sortItemSelected);
				let wordList1 = [];
				let conditions = [];

				if (this.sortItemSelected[0]) {
					conditions.push("word LIKE '%" + inputWord + "%'");
				}
				if (this.sortItemSelected[1]) {
					conditions.push("meaning LIKE '%" + inputWord + "%'");
				}
				if (this.sortItemSelected[2]) {
					conditions.push("example LIKE '%" + inputWord + "%'");
				}
				if (this.sortItemSelected[3]) {
					conditions.push("note LIKE '%" + inputWord + "%'");
				}

				let conditionStr = conditions.join(' OR ');
				console.log(conditionStr);
				plus.sqlite.selectSql({
					name: 'first',
					sql: `SELECT id, word FROM wordtable WHERE ${conditionStr}`,
					success: (e) => {
						if (e.length > 0) {
							let newWordList = e.map(item => ({
								wordId: item.id,
								word: item.word
							}));
							this.wordList = newWordList;
						}
					},
				});
			},
			goToWordDetail(wordId) {
				uni.navigateTo({
					url: '/pages/word/wordDetail?wordId=' + wordId
				});
			},
			selected(index) {
				this.sortItemSelected[index] = !this.sortItemSelected[index];
			},
		}
	}
</script>

<style>
	* {
		--blue-lighten-5: #E3F2FD;
		--blue-lighten-4: #BBDEFB;
		--blue-lighten-3: #90CAF9;
		--blue-lighten-2: #64B5F6;
		--blue-lighten-1: #42A5F5;
		--blue-darken-1: #1E88E5;
		--blue-darken-2: #1976D2;
		--blue-darken-3: #1565C0;
		--blue-darken-4: #0D47A1;
		--blue-accent-1: #82B1FF;
		--blue-accent-2: #448AFF;
		--blue-accent-3: #2979FF;
		--blue-accent-4: #2962FF;

		--uni-font-size-sm: 12px;
		--uni-font-size-base: 14px;
		--uni-font-size-lg: 20px;
		--uni-font-size-sp: 50px;

		--devider-color: #f3f3f3;
	}

	body {
		display: flex;
		flex-direction: column;
	}

	.blank {
		height: calc(var(--status-bar-height)*2.5);
	}

	.search-box {
		display: flex;
		flex-direction: row;
		align-items: center;
		width: 100vw;
		margin-bottom: calc(var(--status-bar-height) *.7);
		height: calc(var(--uni-font-size-base) * 1.5);
	}

	.search-bar {
		display: flex;
		height: 100%;
		width: 82%;
		margin-left: 3%;
		padding: calc(var(--uni-font-size-base) *.5);
		background-color: aliceblue;
		border-radius: 5px;
	}

	.back-btn {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		height: 100%;
		width: 15%;
	}

	.sort-bar {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-evenly;
		width: 96vw;
		margin-left: 0;
		padding: calc(var(--uni-font-size-base) *.5);
		height: calc(var(--uni-font-size-base) * 2);
		/* background-color: var(--blue-lighten-5); */
	}

	.sort-bar-btn {
		display: flex;
		justify-content: center;
		align-items: center;
		border-radius: 5px;
		height: 100%;
		width: 20%;
		/* border: 2px solid var(--blue-lighten-5); */
		background-color: var(--blue-lighten-5);
	}

	.sort-bar-btn-selected {
		background-color: var(--blue-darken-1);
		color: #fff;
	}

	.word-box {
		display: flex;
		flex-direction: column;
	}

	.word-item {
		margin-left: 3vw;
		padding-block: 1vh;
		border-bottom: 1px solid var(--devider-color);
	}

	.word-item:hover {
		color: var(--blue-darken-1);
	}
</style>