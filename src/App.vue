<template>
  <div class="konva-demo">
    <div class="list">
      <div class="canvas-container item left">
        <img
          :src="imgUrl"
          id="originImg"
          ref="originImg"
          style="display: none"
          alt=""
        />
        <div id="container" ref="container"></div>
      </div>
      <div class="item right"></div>
    </div>
    <div>
      <button @click="draggable('move')">移动</button>
      <button @click="draggable('drawRect')">绘图</button>
    </div>
  </div>
</template>
<script>
import $ from 'jquery'
export default {
  data() {
    return {
      imgUrl: require('@/assets/test.jpg'),
      imgWidth: 0,
      imgHeight: 0,
      pointServeList: [
        {
          point: [326, 264, 426, 264, 426, 364, 326, 364],
          text: 'test',
        },
      ],
      pointLocalList: [],
      localPointList: [],
      moveX: 0,
      moveY: 0,
      isRect: false,
      scaleBy: 1.01,
      newScale: 1,
      stageX: 0,
      stageY: 0,
    }
  },
  methods: {
    drawRect(pointStart, pointEnd) {
      if (this.isDraw) {
        this.layer.removeChildren()
        this.localPointList.forEach((item) => {
          let rect = new Konva.Rect(item.attrs)
          // rect.on('click', this.handleRectClick)
          rect.on('mouseenter', () => {
            this.handleRectMouseEnter()
          })
          rect.on('mouseleave', () => {
            this.handleRectMouseLeave()
          })
          this.layer.add(rect)
          this.stage.add(this.layer)
        })
      }
      let width = Math.abs(pointEnd[0] - pointStart[0])
      let height = Math.abs(pointEnd[1] - pointStart[1])
      if (!width || !height || width < 10 || height < 10) {
        return
      }
      console.log(pointStart[1] - this.stageY)
      let rect = new Konva.Rect({
        x: Math.floor((pointStart[0] - this.stageX) / this.newScale),
        y: Math.floor((pointStart[1] - this.stageY) / this.newScale),
        width: Math.floor(width / this.newScale),
        height: Math.floor(height / this.newScale),
        fill: 'rgba(0,0,0,0.6)',
        stroke: 'black',
        strokeWidth: 1,
        draggable: true,
        name: 'rect'
      })
      if (this.isPush) {
        rect.on('mouseenter', () => {
          this.handleRectMouseEnter()
        })
        rect.on('mouseleave', () => {
          this.handleRectMouseLeave()
        })
        this.localPointList.push(rect)
      }
      this.layer.add(rect)
      this.stage.add(this.layer)
      this.isPush = false
      // console.log(this.layer.getChildren())
    },
    initImg() {
      let imageObj = new Image()
      imageObj.onload = () => {
        let img = new Konva.Image({
          x: 0,
          y: 0,
          image: imageObj,
          width: this.imgWidth,
          height: this.imgHeight,
          // draggable: true,
        })
        this.layerImg.add(img)
        this.layerImg.batchDraw()
      }
      imageObj.src = this.imgUrl
    },
    handleMouseDown(e) {
      console.log(e)
      this.startX = e.evt.offsetX - this.moveX
      this.startY = e.evt.offsetY - this.moveY
      this.isDraw = true
    },
    handleMouseMove(e) {
      this.endX = e.evt.offsetX - this.moveX
      this.endY = e.evt.offsetY - this.moveY
      if (this.endX < this.startX) {
        this.isDraw = false
      }
      if (this.isDraw && !this.isMove && !this.isRect) {
        this.drawRect([this.startX, this.startY], [this.endX, this.endY])
      }
    },
    handleMouseUp(e) {
      this.endX = e.evt.offsetX - this.moveX
      this.endY = e.evt.offsetY - this.moveY
      this.isPush = true
      if (this.isDraw && !this.isMove && !this.isRect) {
        this.drawRect([this.startX, this.startY], [this.endX, this.endY])
      }
      this.isDraw = false
    },
    handleeWheel(e) {
      e.evt.preventDefault()
      let oldScale = this.stage.scaleX()
      let mousePointTo = {
        x:
          this.stage.getPointerPosition().x / oldScale -
          this.stage.x() / oldScale,
        y:
          this.stage.getPointerPosition().y / oldScale -
          this.stage.y() / oldScale,
      }

      this.newScale =
        e.evt.deltaY > 0 ? oldScale * this.scaleBy : oldScale / this.scaleBy
      this.stage.scale({ x: this.newScale, y: this.newScale })

      let newPos = {
        x:
          -(
            mousePointTo.x -
            this.stage.getPointerPosition().x / this.newScale
          ) * this.newScale,
        y:
          -(
            mousePointTo.y -
            this.stage.getPointerPosition().y / this.newScale
          ) * this.newScale,
      }
      this.stage.position(newPos)
      console.log(this.stage)
      console.log(this.newScale)
      // this.moveX = newPos.x
      // this.moveY = newPos.y
      // this.stage.setAttrs({...newPos, newScale})
      this.stageX = this.stage.getAttr('x')
      this.stageY = this.stage.getAttr('y')
      this.stage.batchDraw()
    },
    handleClick(e) {
      // console.log(e.target)
      // do nothing if clicked NOT on our rectangles
      if (e.target === this.stage) {
        this.stage.find('Transformer').destroy()
        this.layer.draw()
        return
      }
      if (!e.target.hasName('rect')) {
        return
      }
      // remove old transformers
      // TODO: we can skip it if current rect is already selected
      this.stage.find('Transformer').destroy()

      // create new transformer
      this.tr = new Konva.Transformer()
      this.layer.add(this.tr)
      this.tr.nodes([e.target])
      this.layer.draw()
    },
    draggable(data) {
      if (data === 'drawRect') {
        this.stage.setAttr('draggable', false)
      } else {
        this.stage.setAttr('draggable', true)
      }
      this.isMove = this.stage.getAttr('draggable')
    },
    pointToLocal(list) {
      list.forEach((item) => {
        this.isPush = true
        this.drawRect(
          [item.point[0], item.point[1]],
          [item.point[4], item.point[5]]
        )
      })
    },
    handleRectClick(rect) {
      this.layer.add(this.tr)
      this.tr.nodes([rect])
      this.layer.draw()
    },
    handleRectMouseEnter() {
      this.stage.off('mousedown')
      this.stage.off('mousemove')
      this.stage.off('mouseup')
    },
    handleRectMouseLeave() {
      this.stage.on('mousedown', (e) => {
        this.handleMouseDown(e)
      })
      this.stage.on('mousemove', (e) => {
        this.handleMouseMove(e)
      })
      this.stage.on('mouseup', (e) => {
        this.handleMouseUp(e)
      })
    },
  },
  mounted() {
    this.$refs.originImg.onload = () => {
      this.originWidth = this.$refs.originImg.naturalWidth
      this.originHeight = this.$refs.originImg.naturalHeight
      this.imgWidth = $('#originImg').width()
      this.imgHeight = $('#originImg').height()
      this.stage = new Konva.Stage({
        container: 'container',
        width: this.imgWidth,
        height: this.imgHeight,
        draggable: false,
        dragBoundFunc: (option) => {
          this.moveX = option.x
          this.moveY = option.y
          return option
        },
      })
      this.layer = new Konva.Layer()
      this.layerImg = new Konva.Layer()
      this.layerIcon = new Konva.Layer()
      this.stage.add(this.layerImg)
      this.stage.add(this.layerIcon)
      // this.tr = new Konva.Transformer()
      this.initImg()
      this.pointToLocal(this.pointServeList)
      this.stage.on('mousedown', (e) => {
        this.handleMouseDown(e)
      })
      this.stage.on('mousemove', (e) => {
        this.handleMouseMove(e)
      })
      this.stage.on('mouseup', (e) => {
        this.handleMouseUp(e)
      })
      this.stage.on('wheel', (e) => {
        this.handleeWheel(e)
      })
      this.stage.on('click tap', (e) => {
        this.handleClick(e)
      })
    }
  },
}
</script>
<style>
.konva-demo .list {
  display: flex;
  height: 600px;
  width: 100%;
}
.item {
  border: 1px solid #ccc;
  position: relative;
  flex: 1;
}
.left {
  margin-right: 20px;
}
#originImg {
  max-width: 100%;
  max-height: 100%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, 50%);
}
.canvas-container {
  position: relative;
}
#container {
  max-width: 100%;
  max-height: 100%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
