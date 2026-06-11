<template>
	<view class="container">
		<!-- 顶部粘贴区域 -->
		<view class="card">
			<view class="card-title">输入文本</view>
			<textarea
				v-model="inputText"
				placeholder="粘贴英文文本，关键词汇后加括号标注翻译&#10;例如：expanded（拓展）"
				class="textarea"
				:maxlength="-1"
			></textarea>
			<view class="button-group">
				<button class="btn btn-primary" @click="parseText">解析文本</button>
				<button class="btn btn-secondary" @click="clearText">清空</button>
			</view>
		</view>

		<!-- 解析后的文本显示区域 -->
		<view v-if="sentences.length > 0" class="card">
			<view
				class="text-content"
			>
				<text
					v-for="(sentence, sentenceIndex) in sentences"
					:key="sentenceIndex"
					:class="[
						'sentence-span',
						highlightedSentenceIndex === sentenceIndex ? 'highlighted' : ''
					]"
					:data-sentence-index="sentenceIndex"
					@longpress="handleSentenceLongPress(sentenceIndex, $event)"
				>
					<template v-for="(word, wordIndex) in sentence.words" :key="wordIndex">
						<text v-if="word.type === 'normal'">{{ word.text }}</text>
						<text 
							v-else 
							class="keyword-word"
							@click.stop="toggleWordTranslation(sentenceIndex, wordIndex)"
						>{{ word.word }}</text>
						<text v-if="word.showTranslation" class="keyword-translation">（{{ word.translation }}）</text>
					</template>
					<text> </text>
				</text>
			</view>
		</view>

		<!-- 自定义长按菜单 -->
		<view v-if="showMenu" class="custom-menu" :style="{ top: menuPosition.y + 'px', left: menuPosition.x + 'px' }">
			<view class="menu-item" @click="handleRead">
				<text class="menu-icon">🔊</text>
				<text class="menu-text">朗读</text>
			</view>
			<view class="menu-item" @click="handleCopy">
				<text class="menu-icon">📋</text>
				<text class="menu-text">复制</text>
			</view>
		</view>

		<!-- 遮罩层 -->
		<view v-if="showMenu" class="mask" @click="hideMenu"></view>
	</view>
</template>

<script>
export default {
	data() {
			return {
				inputText: 'Midway（中途） through, I came up with a new study idea. I can have AI organize（整理） my scattered（零散的） verbal thoughts into coherent（通顺的）, well-structured and logically rigorous（严谨的） writing, which can even serve as model reading passages directly.\n \nEarlier in the day, I watched a video about how one\'s native language regulates（调节） emotions. It introduced a useful method: when something upsets（使心烦） you deeply, translate the whole story into English and recite（复述） it fully. This process engages（调动） the left brain, helping you step out of subjective（主观的） emotions and view the matter rationally（理性地） from a neutral（中立的） third-person perspective（视角）. As you sort out（梳理）, translate and retell everything, negative feelings will gradually fade away（逐渐消散）.\n \nInspired by this principle（原理）, I devised（想出） a new learning approach（方法）. I will copy the polished（优化后的） reflections（感悟） edited by AI and translate them into English. Then I will read each sentence, compare the two versions and check if I can fully understand the translated text.\n \nI used to memorize English words rigidly（死板地） by only learning dictionary definitions（释义）, without knowing how to use them flexibly（灵活地） in real life. This new method, however, combines vocabulary and sentence patterns（句式） with my real-life reflections. It allows me to deeply grasp（理解） the practical usage（实际用法） and connotations（深层含义） of words, which I believe is highly efficient（高效的）.\n \nStill, I feel torn（纠结的） about one thing: I cannot decide whether I should keep a dedicated（专门的） record of all the new words I learn. Listing them one by one would be an enormous（庞大的） task. I have been in this situation many times before. I would pile up（堆砌） loads of study tasks and look extremely diligent（勤奋的） to others, yet I could never stick to（坚持） them for more than a few days before giving up（半途而废）.\n \nI also reflected repeatedly on the purpose of learning English now. I have long finished the Gaokao and all other standard examinations（应试考试） in life. Technically（按理说）, I have no urgent（迫切的） need to practice oral English or improve my language skills, so it seems unnecessary to spend time on it.\n \nBut while sorting out these thoughts, I gained a new insight（感悟）: life\'s endless possibilities（可能性） are expanded（拓展） bit by bit through our own efforts. Accumulation（积累） that seems useless today will most likely prove valuable someday in the future. I may not know exactly when these efforts will pay off（起作用）, but I am certain they will never go to waste（白费）.\n \nMore importantly, this way of learning is reshaping（重塑） my way of thinking. I have kept a daily journal and done voice reviews（复盘） of my life for 44 days now. Though it feels like a very long time to me, the actual span（时长） is not that lengthy（长久的）. Even so, I can clearly notice subtle（潜移默化的） yet steady progress in my mindset（心态）, way of thinking and expressive ability（表达能力）.',
				sentences: [],
				showMenu: false,
				menuPosition: { x: 0, y: 0 },
				selectedSentence: '',
				highlightedSentenceIndex: -1
			}
		},
	onLoad() {
		// 监听文本选择变化（H5）
		// #ifdef H5
		document.addEventListener('selectionchange', this.handleSelectionChange)
		// #endif
		
		// 页面加载时自动解析默认文本
		if (this.inputText) {
			this.sentences = this.parseTextIntoSentences(this.inputText)
		}
	},
	onUnload() {
		// #ifdef H5
		document.removeEventListener('selectionchange', this.handleSelectionChange)
		// #endif
	},
	methods: {

		parseText() {
			if (!this.inputText.trim()) {
				uni.showToast({ title: '请输入文本', icon: 'none' })
				return
			}

			uni.showLoading({ title: '解析中...' })
			this.sentences = this.parseTextIntoSentences(this.inputText)
			uni.hideLoading()

			if (this.sentences.length > 0) {
				uni.showToast({ title: '解析成功', icon: 'success' })
			}
		},

		parseTextIntoSentences(text) {
			const sentences = []
			const pattern = /([^.!?。！？,，]+[.!?。！？,，])/g
			
			let match
			while ((match = pattern.exec(text)) !== null) {
				const content = match[1].trim()
				const words = this.parseSentenceIntoWords(content)
				sentences.push({
					text: content,
					originalText: content,
					words: words,
					start: match.index,
					end: match.index + match[1].length
				})
			}
			
			return sentences
		},
		
		parseSentenceIntoWords(sentence) {
			const words = []
			const keywordPattern = /(\w+)[（(]([^）)]+)[）)]/g
			
			let lastIndex = 0
			let match
			
			while ((match = keywordPattern.exec(sentence)) !== null) {
				if (match.index > lastIndex) {
					words.push({
						type: 'normal',
						text: sentence.substring(lastIndex, match.index)
					})
				}
				words.push({
					type: 'keyword',
					word: match[1],
					translation: match[2],
					showTranslation: false
				})
				lastIndex = match.index + match[0].length
			}
			
			if (lastIndex < sentence.length) {
				words.push({
					type: 'normal',
					text: sentence.substring(lastIndex)
				})
			}
			
			return words
		},
		
		toggleWordTranslation(sentenceIndex, wordIndex) {
			if (this.sentences[sentenceIndex] && this.sentences[sentenceIndex].words[wordIndex]) {
				this.sentences[sentenceIndex].words[wordIndex].showTranslation = 
					!this.sentences[sentenceIndex].words[wordIndex].showTranslation
			}
		},

		clearText() {
			this.inputText = ''
			this.sentences = []
		},

		toggleSentenceDetails(index) {
			if (this.sentences[index].showDetails) {
				this.sentences[index].showDetails = false
			} else {
				this.sentences[index].showDetails = true
			}
		},

		handleSentenceLongPress(sentenceIndex, event) {
			// #ifdef H5
			// H5端允许系统选词
			setTimeout(() => {
				const selection = window.getSelection()
				if (selection && selection.toString().trim()) {
					this.handleSelectionChange()
				} else {
					// 如果没有选中文本，使用句子索引
					this.setHighlightByIndex(sentenceIndex)
				}
			}, 300)
			// #endif
			
			// #ifdef APP-PLUS || MP-WEIXIN
			this.setHighlightByIndex(sentenceIndex)
			// 获取触摸位置用于显示菜单
			const touch = event.touches ? event.touches[0] : event
			this.showMenuAtPosition(touch.clientX, touch.clientY)
			// #endif
		},
		
		setHighlightByIndex(sentenceIndex) {
			const sentence = this.sentences[sentenceIndex]
			if (sentence) {
				this.highlightedSentenceIndex = sentenceIndex
				this.selectedSentence = sentence.text.replace(/[（(][^）)]+[）)]/g, '')
			}
		},

		// H5端监听选区变化
		handleSelectionChange() {
			// #ifdef H5
			const selection = window.getSelection()
			if (selection && selection.toString().trim()) {
				const selectedText = selection.toString().trim()
				const range = selection.getRangeAt(0)
				
				// 获取选区位置用于显示菜单
				const rect = range.getBoundingClientRect()
				const menuX = rect.left + rect.width / 2
				const menuY = rect.top - 50

				// 通过DOM元素查找选中的句子索引
				const sentenceIndex = this.findSentenceIndexByElement(range)

				// 智能扩展到句子
				this.expandToSentence(selectedText, menuX, menuY, sentenceIndex)
			}
			// #endif
		},

		// 通过DOM元素查找句子索引
		findSentenceIndexByElement(range) {
			let element = range.startContainer
			if (element.nodeType === Node.TEXT_NODE) {
				element = element.parentElement
			}
			while (element) {
				const index = element.getAttribute('data-sentence-index')
				if (index !== null) {
					return parseInt(index, 10)
				}
				element = element.parentElement
			}
			return -1
		},

		// 将选中的文本扩展到完整句子
		expandToSentence(selectedText, menuX, menuY, sentenceIndex) {
			if (!this.inputText) return

			if (sentenceIndex === -1) {
				sentenceIndex = this.findSentenceContainingWord(selectedText)
			}
			if (sentenceIndex === -1) return

			// 获取完整句子
			const rawSentence = this.sentences[sentenceIndex].text
			
			// 移除括号和其中的中文翻译（复制时不包含翻译）
			this.selectedSentence = rawSentence.replace(/[（(][^）)]+[）)]/g, '')

			// 高亮该句子
			this.highlightedSentenceIndex = sentenceIndex

			// 显示菜单
			this.showMenuAtPosition(menuX, menuY)
		},

		// 查找包含指定单词的句子索引（备用方案）
		findSentenceContainingWord(word) {
			for (let i = 0; i < this.sentences.length; i++) {
				const sentence = this.sentences[i].text
				const regex = new RegExp(`\\b${word}\\b`, 'i')
				if (regex.test(sentence)) {
					return i
				}
			}
			return -1
		},

		// 获取指定位置开始的单词
		getWordAtPosition(text, position) {
			let result = ''
			// 跳过空格
			while (position < text.length && /\s/.test(text[position])) {
				position++
			}
			// 收集字母字符
			while (position < text.length && /[a-zA-Z]/.test(text[position])) {
				result += text[position]
				position++
			}
			return result
		},

		showMenuAtPosition(x, y) {
			const systemInfo = uni.getSystemInfoSync()
			const screenWidth = systemInfo.windowWidth

			const menuWidth = 150
			let menuX = x - menuWidth / 2
			let menuY = y - 120

			if (menuX < 10) menuX = 10
			if (menuX + menuWidth > screenWidth - 10) menuX = screenWidth - menuWidth - 10
			if (menuY < 10) menuY = y + 30

			this.menuPosition = { x: menuX, y: menuY }
			this.showMenu = true
		},

		hideMenu() {
			this.showMenu = false
			this.highlightedSentenceIndex = -1
			
			// #ifdef H5
			// 清除系统选区
			window.getSelection()?.removeAllRanges()
			// #endif
		},

		handleRead() {
			this.hideMenu()

			// #ifdef H5
			if ('speechSynthesis' in window) {
				const utterance = new SpeechSynthesisUtterance(this.selectedSentence)
				utterance.lang = 'en-US'
				speechSynthesis.speak(utterance)
				uni.showToast({ title: '正在朗读...', icon: 'none' })
			} else {
				uni.showToast({ title: '浏览器不支持语音合成', icon: 'none' })
			}
			// #endif

			// #ifdef APP-PLUS
			plus.speech.startSpeak({
				text: this.selectedSentence,
				lang: 'en_US'
			}, () => {
				uni.showToast({ title: '正在朗读...', icon: 'none' })
			}, (e) => {
				uni.showToast({ title: '朗读失败', icon: 'none' })
			})
			// #endif

			// #ifdef MP-WEIXIN
			uni.showToast({ title: '小程序暂不支持语音朗读', icon: 'none' })
			// #endif
		},

		handleCopy() {
			this.hideMenu()

			uni.setClipboardData({
				data: this.selectedSentence,
				success: () => {
					uni.showToast({ title: '已复制到剪贴板', icon: 'success' })
				},
				fail: () => {
					uni.showToast({ title: '复制失败', icon: 'none' })
				}
			})
		}
	}
}
</script>

<style scoped>
.container {
	min-height: 100vh;
	background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
	padding: 30rpx;
	padding-top: calc(env(safe-area-inset-top) + 30rpx);
}

.card {
	background-color: #FFFFFF;
	border-radius: 24rpx;
	padding: 40rpx;
	margin-bottom: 30rpx;
	box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.12);
}

.card-title {
	font-size: 32rpx;
	font-weight: 600;
	color: #1a1a1a;
	margin-bottom: 24rpx;
	display: flex;
	align-items: center;
}

.card-title::before {
	content: '';
	width: 6rpx;
	height: 32rpx;
	background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
	border-radius: 3rpx;
	margin-right: 16rpx;
}

.textarea-wrapper {
	position: relative;
}

.textarea {
	width: 100%;
	min-height: 200rpx;
	font-size: 28rpx;
	line-height: 1.6;
	color: #333333;
	padding: 24rpx;
	background-color: #F5F7FA;
	border-radius: 16rpx;
	border: 2rpx solid #E8ECF0;
	transition: all 0.3s ease;
	box-sizing: border-box;
}

.textarea:focus {
	border-color: #667eea;
	background-color: #FFFFFF;
}

.button-group {
	display: flex;
	gap: 20rpx;
	margin-top: 24rpx;
}

.btn {
	flex: 1;
	height: 88rpx;
	line-height: 88rpx;
	border-radius: 44rpx;
	font-size: 30rpx;
	font-weight: 500;
	border: none;
	transition: all 0.3s ease;
}

.btn-primary {
	background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
	color: #FFFFFF;
	box-shadow: 0 4rpx 16rpx rgba(102, 126, 234, 0.4);
}

.btn-primary:active {
	transform: scale(0.98);
	box-shadow: 0 2rpx 8rpx rgba(102, 126, 234, 0.3);
}

.btn-secondary {
	background-color: #F5F7FA;
	color: #666666;
	border: 2rpx solid #E8ECF0;
}

.btn-secondary:active {
	background-color: #E8ECF0;
}

.text-display-area {
	background-color: #FFFFFF;
	border-radius: 24rpx;
	padding: 48rpx;
	box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.12);
}

.text-content {
	font-size: 32rpx;
	line-height: 2;
	color: #333333;
	word-break: normal;
	white-space: normal;
	overflow-wrap: break-word;
	text-align: justify;
	text-justify: distribute-all-lines;
	text-align-last: justify;
	-webkit-text-align-last: justify;
	user-select: text;
	-webkit-user-select: text;
}

.sentence-span {
	display: inline;
	white-space: normal;
}

.keyword-word {
	color: #667eea;
	font-weight: 500;
	text-decoration: underline;
	text-decoration-color: rgba(102, 126, 234, 0.4);
	text-decoration-style: dashed;
	cursor: pointer;
	transition: all 0.2s ease;
}

.keyword-word:active {
	color: #764ba2;
	text-decoration-color: rgba(118, 75, 162, 0.6);
}

.keyword-translation {
	color: #FF6B6B;
	font-weight: 500;
	margin-left: 8rpx;
	font-size: 28rpx;
	animation: fadeIn 0.2s ease;
}

@keyframes fadeIn {
	from {
		opacity: 0;
		transform: translateY(-4rpx);
	}
	to {
		opacity: 1;
		transform: translateY(0);
	}
}

.highlighted {
	background-color: rgba(102, 126, 234, 0.15);
	border-radius: 8rpx;
	padding: 4rpx 8rpx;
	margin: -4rpx -8rpx;
}

.highlighted .keyword-word {
	color: #667eea;
	background-color: transparent;
}

.custom-menu {
	position: fixed;
	background-color: #FFFFFF;
	border-radius: 20rpx;
	padding: 16rpx 0;
	box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.18);
	z-index: 9999;
	display: flex;
	flex-direction: row;
	min-width: 300rpx;
}

.menu-item {
	flex: 1;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 20rpx 30rpx;
	transition: background-color 0.2s;
}

.menu-item:active {
	background-color: #F5F5F5;
}

.menu-icon {
	font-size: 40rpx;
	margin-bottom: 8rpx;
}

.menu-text {
	font-size: 24rpx;
	color: #666666;
}

.mask {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 9998;
}
</style>
