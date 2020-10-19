<template>
    <div class="pdf-viewer">
        <div class="pdfViewer"></div>
        <div class="pdf-viewer__screen"></div>
    </div>
</template>
<script>
import { getDocument, GlobalWorkerOptions } from "pdfjs-dist/lib/pdf";
import PdfjsWorker from "pdfjs-dist/lib/pdf.worker";
import { EventBus, PDFViewer, PDFLinkService } from "pdfjs-dist/web/pdf_viewer";
import "pdfjs-dist/web/pdf_viewer.css";

setPdfWorker(PdfjsWorker);

export function setPdfWorker(workerSrcOrClass) {
  if (typeof window !== "undefined") delete window.pdfjsWorker;
  delete GlobalWorkerOptions.workerSrc;
  delete GlobalWorkerOptions.workerPort;

  if (typeof workerSrcOrClass === "string") {
    GlobalWorkerOptions.workerSrc = workerSrcOrClass;
  } else if (typeof workerSrcOrClass === "function") {
    GlobalWorkerOptions.workerPort = workerSrcOrClass();
  } else if (workerSrcOrClass instanceof Worker) {
    GlobalWorkerOptions.workerPort = workerSrcOrClass;
  } else if (typeof window !== "undefined" && workerSrcOrClass) {
    window.pdfjsWorker = workerSrcOrClass;
  }
}

async function PDFViewerRander(url,ownerDocument,container,eventBus,linkService){
    const pdfDocument = await getDocument({ url, ownerDocument }).promise
    console.log(pdfDocument)

    const viewer = new PDFViewer({
        container: container,
        eventBus,
        enhanceTextSelection: true,
        removePageBorders: true,
        linkService
    })
    console.log(viewer)

    linkService.setDocument(pdfDocument)
    linkService.setViewer(viewer)
    viewer.setDocument(pdfDocument)

    return {
        PDFDocument: pdfDocument,
        PDFViewer: viewer
    }
}

function findParent(params) {
    
}

export default {
    props: {
        url:String
    },
    data(){
        return {
            keyCode: NaN,
            isGraping: false
        }
    },
    beforeCreate () {
        this.eventBus = new EventBus()
        this.linkService = new PDFLinkService({ eventBus: this.eventBus })
    },
    async mounted () {
        ///currentScale
        const { PDFDocument,PDFViewer } = await PDFViewerRander(
            this.url,
            this.$el.ownerDocument,
            this.$el,
            this.eventBus,
            this.linkService
        )
        window.PDFDocument = this.$$PDFDocument = PDFDocument
        window.PDFViewer = this.$$PDFViewer = PDFViewer

        window.addEventListener('keydown', this.onKeydown)
        window.addEventListener('keyup', this.onKeyup)
        window.addEventListener('mousedown', this.onMouseDown)
        window.addEventListener('mouseup', this.onMouseUp)
        window.addEventListener('mousemove', this.onMouseMove)
    },
    destroyed () {
        window.removeEventListener('keydown', this.onKeydown)
        window.removeEventListener('keyup', this.onKeyup)
        window.removeEventListener('mousedown', this.onMouseDown)
        window.removeEventListener('mouseup', this.onMouseUp)
        window.removeEventListener('mousemove', this.onMouseMove)
    },
    methods: {
        onKeydown($event){
            this.keyCode = $event.keyCode
            if(this.keyCode != 32) return
            this.$el.classList.add('pdf-viewer--can-grab')
            $event.preventDefault()
            $event.stopPropagation()
        },
        onKeyup($event){
            if(this.keyCode != 32) return
            this.$el.classList.remove('pdf-viewer--can-grab')
        },
        onMouseDown($event){
            if(this.keyCode != 32) return
            if($event.target.className != 'pdf-viewer__screen') return
            this.$el.classList.add('pdf-viewer--grabing')
            this.isGraping = true
        },
        onMouseUp($event){
            if(this.keyCode != 32) return
            this.$el.classList.remove('pdf-viewer--grabing')
            this.isGraping = false
        },
        onMouseMove($event){
            if(!this.isGraping) return

            window.scrollTo(
                window.scrollX-=$event.movementX,
                window.scrollY-=$event.movementY,
            )
        }
    },
    watch: {
        async url(val){
            const { PDFDocument,PDFViewer } = await PDFViewerRander(
                val,
                this.$el.ownerDocument,
                this.$el,
                this.eventBus,
                this.linkService
            )
            window.PDFDocument = this.$$PDFDocument = PDFDocument
            window.PDFViewer = this.$$PDFViewer = PDFViewer
        }
    }
}
</script>
<style lang="scss" scoped>
.pdf-viewer{
    position: relative;
    overflow: visible;
    &--can-grab{
        cursor:-webkit-grab;
        .pdf-viewer__screen{
            opacity: 0;
            visibility: visible;
        }
    }
    &--grabing{
        cursor:-webkit-grabbing;
    }
    &__screen{
        position: absolute;
        top:0;
        left: 0;
        bottom: 0;
        right: 0;
        zoom:1;
        z-index: 998;
        background-color: rgba(0,0,0,.8);
        opacity: 0;
        visibility: hidden;
    }
}
</style>
<style lang="scss">
.textLayer {
  z-index: 2;
  opacity: 1;
  mix-blend-mode: multiply;
  line-height: initial;
}

.annotationLayer {
  position: absolute;
  top: 0;

  z-index: 3;
}

html
  body
  .textLayer
  > div:not(.PdfHighlighter__highlight-layer):not(.Highlight):not(.Highlight-emoji) {
  opacity: 1;
  mix-blend-mode: multiply;
}

.textLayer ::selection {
  background: rgba(252, 232, 151, 1);
  mix-blend-mode: multiply;
}

@media all and (-ms-high-contrast: none), (-ms-high-contrast: active) {
  .textLayer {opacity: 0.5;}
}

/* Internet Explorer support method */
@media all and (-ms-high-contrast: none), (-ms-high-contrast: active) {
  .textLayer {opacity: 0.5 }
}

/* Microsoft Edge Browser 12+ (All) - @supports method */
@supports (-ms-ime-align:auto) {
  .textLayer {opacity: 0.5 }
}

</style>