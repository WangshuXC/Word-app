<template>
	<view class="word-name-box">
		<view class="divider1" />
		<view class="word-name-container">
			<p class="word-name"><strong>{{word}}</strong></p>
			<view class="btn-box">
				<view class="edit-btn" @click="goToEdit(this.wordId)">纠错</view>
				<view class="delete-btn" @click="deleteWord(this.wordId)">删除</view>
			</view>
		</view>
	</view>

	<view class="word-info-box">
		<view class="word-info" v-for="(item,index) in wordClass" :key="index">
			{{wordClass[index]}} {{wordMeaning[index]}}
		</view>
		<view class="divider2" />
	</view>

	<view class="word-example-box" v-if="wordExampleF.length>0">
		<label for="example" class="label-example">例句</label>
		<view class="word-example" v-for="(item,index) in wordExampleF" :key="index">
			<p class="f">{{wordExampleF[index]}} </p>
			<p class="c">{{wordExampleC[index]}}</p>
		</view>
		<view class="divider2" />
	</view>

	<view class="word-note-box" v-if="wordNote.length>0">
		<label for="note" class="label-example">备注</label>
		<view class="word-note">
			{{wordNote}}
		</view>
		<!-- <textarea class='note' v-model="wordNote"></textarea> -->
		<view class="divider2" />
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wordId: 0,
				word: 'Error',
				wordClass: [],
				wordMeaning: [],
				wordExampleF: [],
				wordExampleC: [],
				wordNote: '',
			}
		},
		onLoad(options) {
			this.openDB();
			this.wordId = options.wordId;
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
					sql: 'SELECT * FROM wordtable WHERE id=' + "'" + this.wordId + "'",
					success: (e) => {
						let newWordClass = e[0].wordclass.split('@@@');
						let newWordMeaning = e[0].meaning.split('@@@');
						let parts = e[0].example.split('@@@');

						let wordExampleF1 = [];
						let wordExampleC1 = [];
						parts.forEach(part => {
							let subParts = part.split('###');
							if (subParts.length === 2) {
								wordExampleF1.push(subParts[0]);
								wordExampleC1.push(subParts[1]);
							}
						});
						this.word = e[0].word;
						this.wordClass = newWordClass;
						this.wordMeaning = newWordMeaning;
						this.wordExampleF = wordExampleF1;
						this.wordExampleC = wordExampleC1;
						this.wordNote = e[0].note;
					},
				});
			},
			deleteWord(wordId) {
				uni.showModal({
					title: '提示',
					content: '确定要删除该单词吗？',
					success: (res) => {
						if (res.confirm) {
							plus.sqlite.executeSql({
								name: 'first',
								sql: "delete from wordtable where id = '" + wordId + "';",
								success: function(e) {
									uni.redirectTo({
										url: '/pages/word/wordBank'
									});
								},
								fail: function(e) {
									plus.nativeUI.alert('删除失败: ' + JSON.stringify(e));
								}
							});
						}
					}
				});
			},
			goToEdit(wordId) {
				uni.redirectTo({
					url: '/pages/word/editWord?wordId=' + wordId
				});
			}
		}
	}
</script>

<style>
	* {
		--font-color-grey: #aaaaaa;
		--border-color: #e2e2e2;
		--devider-color: #f3f3f3;
		--margin-length: 5vw;
		--uni-color-error: #dd524d;
		--uni-color-error-grey: #b4413f;
	}

	.word-name-box {
		width: 100vw;
		min-height: 10vh;
		position: relative;
	}

	.divider1 {
		width: 100%;
		height: 1px;
		background-color: var(--devider-color);
	}

	.word-name-container {
		width: 100%;
		height: auto;
		min-height: calc(10vh - 1px);
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.word-name {
		width: 60vw;
		height: auto;
		font-size: 30px;
		color: #42A5F5;
		margin-left: var(--margin-length);
		word-wrap: break-word;
		white-space: pre-wrap;
	}

	.btn-box {
		width: 30vw;
		display: flex;
		flex-direction: row;
	}

	.edit-btn {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		height: 40rpx;
		padding-inline: 10rpx;
		font-size: 12px;
		border-radius: 15px;
		border: 1px solid var(--border-color);
		color: var(--font-color-grey);
		margin-right: var(--margin-length);
	}

	.delete-btn {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		height: 40rpx;
		padding-inline: 10rpx;
		font-size: 12px;
		border-radius: 15px;
		border: 1px solid var(--uni-color-error-grey);
		color: var(--uni-color-error);
		margin-right: var(--margin-length);
	}

	.word-info-box {
		width: 100%;
		height: auto;
		display: flex;
		flex-direction: column;
	}

	.word-info {
		margin-left: var(--margin-length);
		margin-right: var(--margin-length);
		margin-bottom: calc(var(--margin-length) *.5);
		font-size: 15px;
	}

	.divider2 {
		width: 100%;
		height: 3vw;
		background-color: var(--devider-color);
	}

	.word-example-box {
		width: 100%;
		height: auto;
		display: flex;
		flex-direction: column;
	}

	.label-example {
		font-size: 15px;
		margin-left: var(--margin-length);
		margin-block: calc(var(--margin-length) *.7);
		color: var(--font-color-grey);
	}

	.word-example {
		margin-left: var(--margin-length);
		margin-right: var(--margin-length);
		margin-bottom: calc(var(--margin-length));

		font-size: 15px;
	}

	.c {
		color: var(--font-color-grey);
	}

	.word-note-box {
		width: 100%;
		height: auto;
		display: flex;
		flex-direction: column;
	}

	.word-note {
		width: 90vw;
		margin-left: var(--margin-length);
		margin-right: var(--margin-length);
		margin-bottom: calc(var(--margin-length));
		word-wrap: break-word;
		white-space: pre-wrap;
		font-size: 15px;
		overflow: auto;
	}
</style>