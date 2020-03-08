<template>
  <input type="file" name="fileUpload" @change="onFileChange" :id="id" :accept="accept">
</template>

<script type="text/babel">
export default {
  name: 'fileupload',
  props: {
    target: {
      type: String
    },
    action: {
      type: String
    },
    id: {
      type: String
    },
    accept: {
      type: String
    }
  },
  data () {
    return {
      file: null,
      acceptedFileTypes: []
    }
  },
  methods: {
    emitter (event, data) {
      this.$emit(event, data)
    },
    updateProgress (oEvent) {
      let vm = this
      if (oEvent.lengthComputable) {
        let percentComplete = Math.round(oEvent.loaded * 100 / oEvent.total)
        vm.emitter('progress', percentComplete)
      } else {
        // Unable to compute progress information since the total size is unknown
        vm.emitter('progress', false)
      }
    },
    onFileChange (e) {
      let vm = this
      this.acceptedFileTypes = this.accept.split(',');

      if (!this.target || this.target === '') {
        console.log('Please provide the target url')
      } else if (!this.action || this.action === '') {
        console.log('Please provide file upload action ( POST / PUT )')
      } else if (this.action !== 'POST' && this.action !== 'PUT') {
        console.log('File upload component only allows POST and PUT Actions')
      } else {
        let files = e.target.files || e.dataTransfer.files

        if (!files.length) {
          return
        };

        /*global FormData XMLHttpRequest:true*/
        /*eslint no-undef: "error"*/

        this.file = files[0]

        // check to see if file type is accepted before uploading
        if (this.acceptedFileTypes.indexOf(this.file.type) == -1) {
          console.log('unsupported filetype used')
          return
        }

        let formData = new FormData()
        formData.append('file', this.file)

        var xhr = new XMLHttpRequest()
        xhr.open(this.action, this.target)

        xhr.onloadstart = function (e) {
          vm.emitter('start', e)
        }
        xhr.onloadend = function (e) {
          vm.emitter('finish', e)
        }
        xhr.upload.onprogress = vm.updateProgress
        xhr.send(formData)
      }
    }
  }
}
</script>