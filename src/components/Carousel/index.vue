<template>
            <button @click="start">start</button>
            <button @click="stop">stop</button>
        <div class="container">
        <!-- 图片列表 -->
            <ul class="ul-img" ref="ulImgDom">
                <li class="li-img">1</li>
                <li class="li-img">2</li>
                <li class="li-img">3</li>
                <li class="li-img">4</li>
                <li class="li-img">5</li>
            </ul>
            <!-- 切换按钮（上一张、下一张 -->
            <div class="prev" @click="prevImg">
                <span>&lt;</span>
            </div>
            <div class="next" @click="nextImg">
                <span>&gt;</span>
            </div>
            <!-- 数字切换按钮 -->
            <div class="num-box">
                <ul class="num-ul" ref="ulNumDom">
                    <li class="li-num" v-for="(item, index) in new Array(len).fill(0)" :key="index" :="index">{{index+1}}</li>
                </ul>
            </div>
        </div>
</template>
<script >
import {  ref, defineComponent, onMounted, onUpdated, reactive } from 'vue'
import axios from 'axios'

export default defineComponent({
    setup(props) {
        const ulImgDom = ref(null)
        const ulNumDom = ref(null)
        let curIndex = ref(0)
        let len = ref(0)
        let numList
        let width = 0
        let mediaRecorder
        onMounted(() => {
            const imgList = ulImgDom.value.getElementsByClassName("li-img")
            len.value = imgList.length
            width = imgList[0].offsetWidth
            navigator.mediaDevices.getUserMedia({audio: true}).then(stream=>{
                let chunks = [];
                const options = {
                    mimeType: 'audio/webm',
                    audioBitsPerSecond: 16000
                };
                mediaRecorder = new MediaRecorder(stream,options);
                
                // 录音开始事件监听（即调用 mediaRecorder.start()时会触发该事件）
                mediaRecorder.onstart = () =>{
                    console.log("record start")
                }
                // 录音可用事件监听，发生于mediaRecorder.stop()调用后，mediaRecorder.onstop 前
                mediaRecorder.ondataavailable = (e) =>{
                    // console.log(e)
                    chunks.push(e.data)
                }
                
                // 录音结束事件监听，发生在mediaRecorder.stop()和 mediaRecorder.ondataavailable 调用后
                mediaRecorder.onstop = () =>{
                    // console.log("record end")
                    // 获取到录音的blob
                    let blob = new Blob(chunks,{type:"audio/webm;codecs=opus"}); 
                    
                    //  将blob转换为file对象，名字可以自己改，一般用于需要将文件上传到后台的情况
                    let file = new window.File([blob],"record.webm");
                    let formData=new FormData();
                    formData.append('file',file,file.name)
                    submitFile(formData)
                    // 将blob转换为地址，一般用于页面上面的回显，这个url可以直接被 audio 标签使用
                    let url = (window.URL || window.webkitURL).createObjectURL(blob);
                }
            })
        })

        onUpdated(() => {
            numList = ulNumDom.value.getElementsByClassName("li-num")
            numList[curIndex.value].style.backgroundColor = "black";
        })
        function prevImg() {
            console.log(99)
            ulImgDom.value.style.transition = "0.3s";
            numList[curIndex.value].style.backgroundColor = ""; // 清空上一个数字按钮样式
            console.log(curIndex.value)
            if (curIndex.value === 0) {
                ulImgDom.value.style.transition = "0s";
                curIndex.value = len.value - 1;
            } else {
                --curIndex.value;
            }
            console.log('curIndex:',curIndex.value)
            console.log('ulImgDom:',ulImgDom.value)

            ulImgDom.value.style.left = `-${curIndex.value * width}px`; // 根据curIndex设置偏移量从而控制图片显示
            numList[curIndex.value].style.backgroundColor = "black";
        }

        function nextImg() {
            ulImgDom.value.style.transition = "0.3s";
            numList[curIndex.value].style.backgroundColor = "";
            console.log(len, curIndex.value);
            if (curIndex.value === len.value - 1) {
                ulImgDom.value.style.transition = "0s";
                curIndex.value = 0;
            } else {
                ++curIndex.value;
            }
            ulImgDom.value.style.left = `-${curIndex.value * width}px`;
            numList[curIndex.value].style.backgroundColor = "black";
        }

        function start() {
            // console.log('start')
            console.log(mediaRecorder)
            mediaRecorder.start()
        }
        function stop() {
            mediaRecorder.stop()
        }

        function submitFile(formData) {
            axios.post('/api/getFile', formData).then(res => {
                const context = res.data.context
                console.log('context:',context)
                if(context.includes('上一张')){
                    prevImg()
                }else if(context.includes('下一张')){
                    nextImg()
                }
            }).catch(err => {
                console.log(err)
            })
        }

        return {
            ulImgDom,
            ulNumDom,
            prevImg,
            nextImg,
            len,
            start,
            stop
        }
    }
})

</script>

<style scoped>
    .container {
    position: relative;
    height: 400px;
    width: 600px;
    margin: 0 auto;
    background-color: gray;
    overflow: hidden;
}

.ul-img {
    position: absolute;
    display: flex;
    width: 4200px;
    height: 400px;
    padding-left: 0px;
    top: 0;
    margin: 0;
    left: 0;
}

.li-img {
    list-style: none;
    width: 600px;
    height: 400px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: aliceblue;
    font-size: 30px;
    font-weight: 800;
    border: 1px solid #ccc;
    padding-left: 0;
    box-sizing: border-box;
}

.prev,
.next {
    position: absolute;
    width: 70px;
    height: 400px;
    display: flex;
    justify-content: center;
    align-items: center;
    top: 0;
}

.prev {
    left: 0;
}

.next {
    right: 0;
}

.prev span,
.next span {
    display: block;
    color: #fff;
    width: 40px;
    height: 40px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.5);
    border-radius: 50%;
    cursor: pointer;
}

.num-box {
    position: absolute;
    left: 50%;
    bottom: 20px;
    transform: translate(-50%, 0);
    z-index: 2;
}

.num-ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
}

.num-ul li {
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background-color: rgba(130, 121, 121, 0.932);
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 9px;
    color: #fff;
    margin: 0 4px;
    cursor: pointer;
    user-select: none;
}
</style>