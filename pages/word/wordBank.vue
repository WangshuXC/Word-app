<template>
	<view class="blank" v-if="wordList.length>1">
		<view class="sort-btn" @click="toggleSortOrder">
			<text v-if="isAscending">排序▲</text>
			<text v-else>排序▼</text>
		</view>
	</view>
	<view class="word-box" v-for="(item,index) in wordList" :key='index' @click="goToWordDetail(item.wordId)">
		<view class="word-name">
			{{item.word}}
		</view>
		<view class="word-meaning">
			{{item.meaning}}
		</view>
		<label for="detail" class="detail">详细 ></label>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wordList: [{
					wordId: 0,
					word: 'Error',
					meaning: '错误',
				}],
				isAscending: true,
			}
		},
		onLoad() {
			this.openDB();
		},
		mounted() {
			this.loadWordData();
		},
		methods: {
			openDB() {
				plus.sqlite.openDatabase({
					name: 'first',
					path: '_doc/data.db',
				});
			},
			loadWordData() {
				plus.sqlite.selectSql({
					name: 'first',
					sql: `SELECT id, word, meaning FROM wordtable ORDER BY word ${this.isAscending ? 'ASC' : 'DESC'}`,
					success: (e) => {
						let newWordList = [];
						for (let index = 0; index < e.length; index++) {
							let wordObj = {
								wordId: e[index].id,
								word: e[index].word,
								meaning: e[index].meaning.split('@@@')[0]
							};
							newWordList.push(wordObj);
						}
						this.wordList = newWordList;
					},
				});
			},
			goToWordDetail(wordId) {
				uni.redirectTo({
					url: '/pages/word/wordDetail?wordId=' + wordId
				});
			},
			toggleSortOrder() {
				this.isAscending = !this.isAscending;
				this.loadWordData()
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

		--border-color: #f3f3f3;
	}

	body {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.blank {
		width: 100vw;
		position: relative;
		height: var(--status-bar-height);
	}

	.sort-btn {
		color: var(--blue-darken-1);
		position: absolute;
		right: 2vw;
	}

	.word-box {
		display: flex;
		flex-direction: column;
		position: relative;
		/* align-items: center; */
		width: 100vw;
		height: auto;
		padding: 2vw;
		border-bottom: 2px solid var(--border-color);
	}

	.word-box:hover {
		background-color: var(--blue-lighten-5);
	}

	.word-name {
		font-size: 25px;
		width: 80%;
		margin-left: 5vw;
		word-wrap: break-word;
		white-space: pre-wrap;
		text-overflow: ellipsis;
	}

	.word-meaning {
		font-size: 15px;
		color: #787878;
		max-width: 80%;
		margin-top: 30rpx;
		margin-left: 5vw;
		margin-bottom: calc(30rpx - 2vw);
	}

	.detail {
		position: absolute;
		right: 5vw;
		font-size: 15px;
		color: var(--blue-darken-1);
		bottom: 30rpx;
	}
</style>