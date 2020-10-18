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

export default {
    props: {
        url:String
    },
    beforeCreate () {
        this.eventBus = new EventBus()
        this.linkService = new PDFLinkService({ eventBus: this.eventBus })
    },
    async mounted () {
        const url = this.url
        const ownerDocument = this.$el.ownerDocument
        const pdfDocument = this.pdfDocument = await getDocument({ url, ownerDocument }).promise
        console.log(pdfDocument)

        const viewer = this.viewer = new PDFViewer({
            container: this.$$wrapper.elm,
            eventBus: this.eventBus,
            enhanceTextSelection: true,
            removePageBorders: true,
            linkService: this.linkService
        })

        this.linkService.setDocument(pdfDocument)
        this.linkService.setViewer(viewer)
        this.viewer.setDocument(pdfDocument)
    },
    render(createElement){
        const wrapper = createElement('div', [createElement("div", "The pdf veiwer fail!")])
        this.$$wrapper = wrapper
        return wrapper
    }
}
</script>