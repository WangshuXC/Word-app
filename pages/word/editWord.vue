<template>
	<view class="word-info-box">
		<label for="word-name" @click="openDB()">单词</label>
		<textarea class="word-name-input" type="text" v-model="wordName" auto-height />

		<label for="word-class">词性及含义</label>
		<view class="word-class-box">
			<view class="word-class" v-for="(item, index) in wordClassIndex" :key="index">
				<picker @change="bindPickerChange" :id="index" :value="index" :range="wordClassList">
					<view class="uni-input">{{ wordClassList[wordClassIndex[index]] }}</view>
				</picker>
				<textarea class="word-class-input" v-model="wordClass[index]" auto-height />
			</view>
			<view class="word-class-btn-box" id='class'>
				<view class="change-word-class" v-if="wordClassBtn[0]" @click="addWordClass">＋</view>
				<view class="change-word-class" v-if="wordClassBtn[1]" @click="subWordClass">-</view>
			</view>
		</view>



		<label for="word-example" @click="selectSQL()">例句</label>
		<view class="word-example-box">
			<view class="word-example" v-for="(item, index) in wordExampleF" :key="index">
				<view class="word-example-input-box">
					<textarea class="word-example-input" id='foreign' auto-height placeholder="输入外文例句"
						v-model="wordExampleF[index]" />
					<textarea class="word-example-input" id='chinese' auto-height placeholder="输入例句翻译"
						v-model="wordExampleC[index]" />
				</view>
			</view>
			<view class="word-example-btn-box" id='example'>
				<view class="change-word-class" v-if="wordExampleBtn[0]" @click="addWordExample">＋</view>
				<view class="change-word-class" v-if="wordExampleBtn[1]" @click="subWordExample">-</view>
			</view>
		</view>

		<label for="word-note" @click="selectSQL()">备注</label>
		<textarea class="word-note-input" type="text" v-model="wordNote" auto-height maxlength="1000" />

		<view class="submit-btn-box">
			<view class="submit-btn" @click="deleteWord(),submit()">
				提交
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wordId: 0,
				wordName: '',
				wordClassList: ['n.', 'pron.', 'adj.', 'adv.', 'vt.', 'vi.', 'conj.',
					'prep.', 'int.', 'vaux.', 'art.', 'num.', 'det.'
				],
				wordClassBtn: [1, 0],
				wordClassIndex: [0],
				wordClass: [],

				wordExampleBtn: [1, 0],
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
					name: 'insert',
					path: '_doc/data.db',
				});
			},
			bindPickerChange(e) {
				const pickerIndex = e.currentTarget.id;
				this.wordClassIndex[pickerIndex] = e.detail.value;
			},
			addWordClass() {
				if (this.wordClassIndex.length + 1 == this.wordClassList.length) {
					this.wordClassBtn[0] = 0;
				}

				if (this.wordClassIndex.length + 1 <= this.wordClassList.length) {
					this.wordClassIndex.push(this.wordClassIndex[this.wordClassIndex.length - 1] + 1);
					this.wordClass.push("");
					this.wordClassBtn[1] = 1;
				}
			},
			subWordClass() {
				if (this.wordClassIndex.length == 2) {
					this.wordClassBtn[1] = 0;
				}

				if (this.wordClassIndex.length == this.wordClassList.length) {
					this.wordClassBtn[0] = 1;
				}

				if (this.wordClassIndex.length > 1) {
					this.wordClassIndex.pop(this.wordClassIndex[this.wordClassIndex.length - 1]);
					this.wordClass.pop(this.wordClass[this.wordClass.length - 1])
				}
			},
			addWordExample() {
				this.wordExampleF.push("");
				this.wordExampleC.push("");
				this.wordExampleBtn[1] = 1;
			},
			subWordExample() {
				if (this.wordExampleF.length == 2) {
					this.wordExampleBtn[1] = 0;
				}
				if (this.wordExampleF.length > 1) {
					this.wordExampleF.pop();
					this.wordExampleC.pop();
				}
			},
			loadWordData() {
				plus.sqlite.selectSql({
					name: 'first',
					sql: 'SELECT * FROM wordtable WHERE id=' + "'" + this.wordId + "'",
					success: (e) => {
						let newWordClass = e[0].wordclasscode.split('@@@');
						newWordClass.pop();
						let newWordMeaning = e[0].meaning.split('@@@');

						let newWordClassNumbers = [];
						for (let index = 0; index < newWordMeaning.length; index++) {
							newWordClassNumbers[index] = parseInt(newWordClass[index], 10);
						}
						newWordClass = newWordClassNumbers;

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
						this.wordName = e[0].word;
						this.wordClassIndex = newWordClass;
						this.wordClass = newWordMeaning;
						this.wordExampleF = wordExampleF1;
						this.wordExampleC = wordExampleC1;
						this.wordNote = e[0].note;
					},
				});
			},
			deleteWord() {
				let toWord = this.wordName;
				plus.sqlite.executeSql({
					name: 'first',
					sql: "delete from wordtable where word = '" + this.wordName + "';",
					success: function(e) {
						uni.redirectTo({
							url: '/pages/word/wordDetail?word=' + toWord
						});
					},
					fail: function(e) {
						plus.nativeUI.alert('删除失败: ' + JSON.stringify(e));
					}
				});
			},
			submit() {
				let meaning = String(this.wordClass.join('@@@')).replace(/'/g, "''");
				let classtype = '';
				let classtypecode = '';
				let example = '';

				for (let index of this.wordClassIndex) {
					classtype += this.wordClassList[index] + '@@@';
					classtypecode += index + '@@@';
				}
				classtype = classtype.slice(0, -3);
				for (let index = 0; index < Math.min(this.wordExampleF.length, this.wordExampleC.length); index++) {
					example += this.wordExampleF[index].replace(/'/g, "''") + '###' + this.wordExampleC[index].replace(
						/'/g, "''") + '@@@';
				}
				example = example.slice(0, -3);

				if (this.wordClass.length === 0) {
					plus.nativeUI.alert('意思存在空值，无法添加');

					return;
				}

				let wordName = this.wordName.replace(/'/g, "''");
				let wordNote = this.wordNote.replace(/'/g, "''");

				plus.sqlite.executeSql({
					name: 'insert',
					sql: `INSERT INTO wordtable (word, meaning, wordclass, wordclasscode, example, note) VALUES ('${wordName}', '${meaning}', '${classtype}', '${classtypecode}', '${example}', '${wordNote}')`,
					success(e) {
						console.log('插入成功');
					},
					fail(e) {
						console.log('插入失败' + e);
					}
				});
			},
			selectSQL() {
				var self = this;
				plus.sqlite.selectSql({
					name: 'first',
					sql: 'select * from word',
					success: function(e) {
						plus.nativeUI.alert('查询SQL语句成功: ' + JSON.stringify(e));
					},
					fail: function(e) {
						plus.nativeUI.alert('查询SQL语句失败: ' + JSON.stringify(e));
					}
				});
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

		/* 边框颜色 */
		--uni-border-color1: #c8c7cc;

		/* 文字基本颜色 */
		--uni-text-color: #333;
		--uni-text-color-inverse: #fff;
		--uni-text-color-grey: #a8d0ff;
		--uni-text-color-placeholder: #808080;
		--uni-text-color-disable: #c0c0c0;

		/* 文字尺寸 */
		--uni-font-size-sm: 12px;
		--uni-font-size-base: 20px;
		--uni-font-size-lg: 25px;
		--uni-font-size-sp: 30px;
	}

	.word-info-box {
		display: flex;
		flex-direction: column;
		margin-left: 5vw;
		width: 95vw;
		height: auto;
	}

	label {
		margin-top: 30rpx;
	}

	.word-name-input {
		width: 87vw;
		height: auto;
		min-height: calc(var(--uni-font-size-sp) + 10rpx);
		font-size: var(--uni-font-size-sp);
		margin-top: 10rpx;
		border: 2px solid var(--blue-lighten-1);
		border-radius: 15px;
		padding: 5px;
		caret-color: var(--blue-lighten-1);
	}

	.word-class-box {
		display: flex;
		flex-direction: column;
		align-content: space-around;
		align-items: center;
		width: 91vw;
		height: auto;
		margin-top: 10rpx;
		font-size: var(--uni-font-size-sp);
		min-height: calc(var(--uni-font-size-sp) + 30rpx);
		/* background-color: var(--blue-lighten-4); */
		border-radius: 15px;
	}

	.word-class {
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		height: auto;
		margin-top: 10rpx;
		min-height: calc(var(--uni-font-size-sp) + 10rpx);
	}

	.uni-input {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		font-size: var(--uni-font-size-base);
		height: 70%;
		width: 15vw;
		padding: 5rpx;
		border-radius: 10px;
		color: var(--uni-text-color-inverse);
		background-color: var(--blue-lighten-1);
	}

	.word-class-input {
		min-height: 70%;
		width: 75%;
		font-size: var(--uni-font-size-base);
		border: 2px solid var(--blue-lighten-1);
		border-radius: 10px;
		padding: 5rpx;
	}

	.word-class-btn-box {
		display: flex;
		flex-direction: row;
		justify-content: space-evenly;

		width: 100%;
		height: auto;
	}

	.change-word-class {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		width: 25%;
		height: calc(var(--uni-font-size-sp) + 10rpx);
		color: var(--uni-text-color-inverse);
		margin-top: 20rpx;
		border-radius: 10px;
		background-color: var(--blue-lighten-1);
	}

	.word-example-box {
		display: flex;
		flex-direction: column;
		align-content: space-around;
		align-items: center;
		width: 91vw;
		height: auto;
		margin-top: 10rpx;
		font-size: var(--uni-font-size-sp);
		min-height: calc(var(--uni-font-size-sp) + 30rpx);
		/* background-color: var(--blue-lighten-4); */
		border-radius: 15px;
	}

	.word-example {
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		height: auto;
		margin-top: 10rpx;
	}

	.word-example-input-box {
		display: flex;
		flex-direction: column;
		min-height: calc((var(--uni-font-size-base) + 10rpx) * 3);
		height: auto;
		width: 100%;
		padding: 15rpx;
		padding-bottom: 25rpx;
		border-radius: 10px;
		background-color: var(--blue-lighten-5);
		margin-bottom: 10rpx;
	}

	.word-example-input {
		height: calc(var(--uni-font-size-base) + 10rpx);
		width: 97%;
		font-size: var(--uni-font-size-base);
		border-radius: 10px;
		padding: 5rpx;
		margin-top: 10rpx;
	}

	.word-example-input#foreign {
		border: 2px solid var(--blue-lighten-3);
	}

	.word-example-input#chinese {
		border: 2px solid var(--blue-lighten-3);
	}

	.word-example-btn-box {
		display: flex;
		flex-direction: row;
		justify-content: space-evenly;

		width: 100%;
		height: auto;

		margin-top: -10rpx;
	}

	.plus-word-example {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		width: 25%;
		height: calc(var(--uni-font-size-sp) + 10rpx);
		color: var(--uni-text-color-inverse);
		margin-top: 20rpx;
		border-radius: 10px;
		background-color: var(--blue-lighten-1);
	}

	.word-note-input {
		width: 87vw;
		height: auto;
		min-height: calc(var(--uni-font-size-lg) + 10rpx);
		font-size: var(--uni-font-size-lg);
		margin-top: 10rpx;
		border: 2px solid var(--blue-lighten-1);
		border-radius: 15px;
		padding: 5px;
		caret-color: var(--blue-lighten-1);
	}

	.submit-btn-box {
		display: flex;
		justify-content: center;
		height: auto;
		width: 91vw;
		margin-block: 60rpx;
	}

	.submit-btn {
		display: flex;
		justify-content: center;
		align-content: center;
		align-items: center;
		width: 30%;
		height: calc(91vw*.15);
		border-radius: 10px;
		font-size: calc(91vw*.1);
		background-color: var(--blue-darken-2);
		color: var(--uni-text-color-inverse);
	}
</style>