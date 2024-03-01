<template>
	<view class="background">
		<view class="word-count">
			<p>总共添加</p>
			<strong class="count">{{wordCount}}</strong>
			<view class="word-bank" @click="wordBankPage()">
				词库
			</view>
		</view>
	</view>
	<view class="control">
		<view class="control-box">
			<view class="word-btn" id='insert' @click='insertPage()'>
				添加
			</view>
			<view class="word-btn" id='search' @click='searchPage()'>
				查询
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				wordCount: 114514,
			}
		},
		onLoad() {
			this.openDB();
			this.updateWordCount();
		},
		onShow() {
			this.updateWordCount();
		},
		methods: {
			//打开数据库
			openDB() {
				plus.sqlite.openDatabase({
					name: 'first',
					path: '_doc/data.db',
				});
			},
			// 执行SQL语句
			updateWordCount() {
				var self = this;
				plus.sqlite.executeSql({
					name: 'first',
					sql: `CREATE TABLE IF NOT EXISTS wordtable (id INTEGER PRIMARY KEY AUTOINCREMENT, word TEXT UNIQUE,meaning TEXT,wordclass TEXT,wordclasscode TEXT,example TEXT,note TEXT);`,
					success: function(e) {
						plus.sqlite.selectSql({
							name: 'first',
							sql: "SELECT COUNT(*) AS count FROM wordtable;",
							success: function(a) {
								self.wordCount = a[0].count;
							},
							fail: function(e) {
								plus.nativeUI.alert('获取数据失败: ' + JSON.stringify(e));
							}
						});
					},
					fail: function(e) {
						plus.nativeUI.alert('创建表table失败: ' + JSON.stringify(e));
					}
				});
			},
			droptable() {
				plus.sqlite.executeSql({
					name: 'first',
					sql: 'drop table wordtable',
					success: function(e) {
						plus.nativeUI.alert('删除表word成功');
					},
					fail: function(e) {
						plus.nativeUI.alert('删除表word失败: ' + JSON.stringify(e));
					}
				});
			},
			selectSQL: function() {
				plus.sqlite.selectSql({
					name: 'first',
					sql: 'select * from wordtable',
					success: function(e) {
						plus.nativeUI.alert('查询SQL语句成功: ' + JSON.stringify(e));
					},
					fail: function(e) {
						plus.nativeUI.alert('查询SQL语句失败: ' + JSON.stringify(e));
					}
				});
			},
			insertPage() {
				uni.navigateTo({
					url: '/pages/word/insertWord'
				});
			},
			searchPage() {
				uni.navigateTo({
					url: '/pages/word/searchWord'
				});
			},
			wordBankPage() {
				uni.navigateTo({
					url: '/pages/word/wordBank'
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

		--uni-color-success: #4cd964;
		--uni-color-warning: #f0ad4e;
		--uni-color-error: #dd524d;

		/* 背景颜色 */
		--blue-lighten-1-grey: #f8f8f8;

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
		--uni-font-size-base: 14px;
		--uni-font-size-lg: 20px;
		--uni-font-size-sp: 50px;
	}

	.background {
		height: 75vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background-color: var(--blue-lighten-1);
	}

	.word-count {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: -20vh;
		font-size: var(--uni-font-size-sp);
		color: #ffffff;
	}

	p {
		font-size: var(--uni-font-size-base);
		color: var(--uni-text-color-grey)
	}

	.word-bank {
		width: auto;
		padding-block: 1px;
		padding-inline: 10px;
		margin-top: 10px;
		border-radius: 15px;
		border: 1px solid #ffffff;
		font-size: var(--uni-font-size-base);
	}

	.control {
		height: 25vh;
		background-color: var(--blue-lighten-1-grey);
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.control-box {
		width: 90vw;
		height: 24vh;
		margin-top: -6vh;

		display: flex;
		align-content: space-between;
		align-items: center;

		border-radius: 10px;
		box-shadow: 0 4px 6px 3px rgba(0, 122, 255, 0.2);
		background-color: #ffffff;
	}

	.word-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 40%;
		height: 25%;
		margin-inline: 5%;
		border-radius: 30px;
		border: 3px solid var(--blue-lighten-1);
		color: var(--blue-lighten-1);
		font-size: var(--uni-font-size-lg);
	}

	.word-btn#insert {
		background-color: var(--blue-lighten-1);
		color: #ffffff
	}
</style>