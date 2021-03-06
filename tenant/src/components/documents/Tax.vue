<template>
  <div>
    <ValidationObserver v-slot="{ invalid, validate }">
      <form name="form" @submit.prevent="validate().then(save)">
        <div>
          <select
            v-model="taxDocument"
            class="rf-select rf-mb-3w"
            id="select"
            name="select"
          >
            <option v-for="d in documents" :value="d" :key="d.key">
              {{ $t(d.key) }}
            </option>
          </select>
        </div>
        <div v-if="taxDocument.key && taxDocument.key === 'other-tax'">
          <div class="rf-input-group">
            <label class="rf-label" for="customText">{{
              $t("custom-text")
            }}</label>
            <input
              v-model="customText"
              class="form-control rf-input validate-required"
              id="customText"
              name="customText"
              placeholder=""
              type="text"
              required
            />
          </div>
        </div>
        <div v-if="taxDocument.key && taxDocument.key === 'my-name'">
          <div class="rf-mb-3w">
            {{ taxDocument.explanationText }}
          </div>
          <div class="rf-mb-3w">
            <FileUpload
              :current-status="fileUploadStatus"
              @add-files="addFiles"
              @reset-files="resetFiles"
            ></FileUpload>
          </div>
          <div class="rf-mb-3w">
            <DocumentInsert
              :allow-list="taxDocument.acceptedProofs"
              :block-list="taxDocument.refusedProofs"
            ></DocumentInsert>
          </div>
        </div>
        <div v-if="taxFiles().length > 0">
          <h5>{{ $t("files") }}</h5>
          <ListItem
            v-for="(file, k) in taxFiles()"
            :key="k"
            :file="file"
            @remove="remove(file)"
          />
        </div>
        <div class="rf-col-12 rf-mb-3w" v-if="taxDocument.key === 'my-name'">
          <validation-provider rules="is" v-slot="{ errors }" class="rf-col-10">
            <div
              class="rf-input-group"
              :class="errors[0] ? 'rf-input-group--error' : ''"
            >
              <input
                type="checkbox"
                id="acceptVerification"
                value="false"
                v-model="acceptVerification"
              />
              <label for="acceptVerification">{{
                $t("accept-verification")
              }}</label>
              <span class="rf-error-text" v-if="errors[0]">{{
                $t(errors[0])
              }}</span>
            </div>
          </validation-provider>
        </div>
        <div class="rf-col-12 rf-mb-5w" v-if="taxDocument">
          <button class="rf-btn" type="submit" :disabled="!taxDocument.key">
            {{ $t("register") }}
          </button>
        </div>
      </form>
    </ValidationObserver>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { DocumentType } from "df-shared/src/models/Document";
import DocumentInsert from "@/components/documents/DocumentInsert.vue";
import FileUpload from "@/components/uploads/FileUpload.vue";
import { mapGetters } from "vuex";
import { UploadStatus } from "../uploads/UploadStatus";
import axios from "axios";
import ListItem from "@/components/uploads/ListItem.vue";
import { User } from "df-shared/src/models/User";
import { DfFile } from "df-shared/src/models/DfFile";
import { DfDocument } from "df-shared/src/models/DfDocument";
import { Guarantor } from "df-shared/src/models/Guarantor";
import { extend } from "vee-validate";
import { is } from "vee-validate/dist/rules";
import { ValidationObserver, ValidationProvider } from "vee-validate";

extend("is", {
  ...is,
  message: "field-required",
  validate: (value) => !!value,
});

@Component({
  components: {
    DocumentInsert,
    FileUpload,
    ListItem,
    ValidationObserver,
    ValidationProvider,
  },
  computed: {
    ...mapGetters({
      user: "userToEdit",
    }),
  },
})
export default class Tax extends Vue {
  user!: User | Guarantor;
  fileUploadStatus = UploadStatus.STATUS_INITIAL;
  files: DfFile[] = [];
  uploadProgress: {
    [key: string]: { state: string; percentage: number };
  } = {};
  taxDocument = new DocumentType();

  acceptVerification = false;
  customText = "";

  mounted() {
    if (this.user.documents !== null) {
      const doc = this.user.documents?.find((d: DfDocument) => {
        return d.documentCategory === "TAX";
      });
      if (doc !== undefined) {
        this.customText = doc.customText || "";
        const localDoc = this.documents.find((d: DocumentType) => {
          return d.value === doc.documentSubCategory;
        });
        if (localDoc !== undefined) {
          this.taxDocument = localDoc;
        }
      }
    }
  }

  addFiles(fileList: File[]) {
    const nf = Array.from(fileList).map((f) => {
      return { name: f.name, file: f };
    });
    this.files = [...this.files, ...nf];
  }

  resetFiles() {
    this.fileUploadStatus = UploadStatus.STATUS_INITIAL;
  }
  save() {
    if (
      !this.taxDocument.key ||
      (this.taxDocument.key === "my-name" && !this.acceptVerification)
    ) {
      return;
    }
    this.uploadProgress = {};
    const fieldName = "documents";
    const formData = new FormData();
    const newFiles = this.files.filter((f) => {
      return !f.id;
    });
    if (newFiles.length) {
      Array.from(Array(newFiles.length).keys()).map((x) => {
        const f: File = newFiles[x].file || new File([], "");
        formData.append(`${fieldName}[${x}]`, f, newFiles[x].name);
      });
    }
    if (this.taxDocument.key === "my-name") {
      formData.append(
        "acceptVerification",
        this.acceptVerification ? "true" : "false"
      );
    }

    formData.append("typeDocumentTax", this.taxDocument.value);

    if (this.taxDocument.key === "other-tax") {
      formData.append("customText", this.customText);
    }

    this.fileUploadStatus = UploadStatus.STATUS_SAVING;
    let url: string;
    if (this.$store.getters.isGuarantor) {
      url = `//${process.env.VUE_APP_API_URL}/api/register/guarantorNaturalPerson/documentTax`;
      formData.append("guarantorId", this.$store.getters.guarantor.id);
    } else {
      url = `//${process.env.VUE_APP_API_URL}/api/register/documentTax`;
    }
    const loader = this.$loading.show();
    axios
      .post(url, formData)
      .then(() => {
        console.log("success");
        this.files = [];
        this.fileUploadStatus = UploadStatus.STATUS_INITIAL;
      })
      .catch(() => {
        console.log("fail");
        this.fileUploadStatus = UploadStatus.STATUS_FAILED;
      })
      .finally(() => {
        this.$store.dispatch("loadUser");
        loader.hide();
      });
  }

  taxFiles() {
    const newFiles = this.files.map((f) => {
      return {
        documentSubCategory: this.taxDocument.value,
        id: f.name,
        name: f.name,
      };
    });
    const existingFiles =
      this.$store.getters.getDocuments?.find((d: DfDocument) => {
        return d.documentCategory === "TAX";
      })?.files || [];
    return [...newFiles, ...existingFiles];
  }

  remove(file: DfFile) {
    if (file.path) {
      const url = `//${process.env.VUE_APP_API_URL}/api/file/${file.id}`;
      const loader = this.$loading.show();
      axios.delete(url).finally(() => {
        this.$store.dispatch("loadUser");
        loader.hide();
      });
    } else {
      this.files = this.files.filter((f: DfFile) => {
        return f.name !== file.name;
      });
    }
  }

  documents: DocumentType[] = [
    {
      key: "my-name",
      value: "MY_NAME",
      explanationText:
        "En joignant mon avis d’imposition, j’accepte que DossierFacile procède à une vérification automatisée de ma fiche d’imposition auprès des services des impôts.\n" +
        "J’ajoute un avis d’imposition à mon nom.",
      acceptedProofs: ["Avis d’imposition de moins de 2 ans"],
      refusedProofs: [
        "Avis d’imposition incomplet (sans la première page)",
        "Tout avis d’imposition plus ancien",
        "Tout autre document justificatif",
      ],
    },
    {
      key: "my-parents",
      value: "MY_PARENTS",
      explanationText:
        "J’ai déclaré être rattaché·e au domicile fiscal de mes parents.",
      acceptedProofs: [],
      refusedProofs: [],
    },
    {
      key: "less-than-year",
      value: "LESS_THAN_YEAR",
      explanationText: "J’ai déclaré être en France depuis moins d’un an.",
      acceptedProofs: [],
      refusedProofs: [],
    },
    {
      key: "other-tax",
      value: "OTHER_TAX",
      explanationText:
        "Afin d’améliorer mon dossier, j’explique ci-dessous pourquoi je ne reçois pas d’avis d’imposition. Mon explication sera ajoutée à mon dossier :",
      acceptedProofs: [],
      refusedProofs: [],
    },
  ];
}
</script>

<style scoped lang="scss"></style>

<i18n>
{
"en": {
"my-name": "Vous avez un avis d’imposition à votre nom",
"my-parents": "Vous êtes rattaché fiscalement à vos parents",
"less-than-year": "Vous êtes en France depuis moins d’un an",
"other-tax": "Autre",
"accept-verification": "J'accepte que DossierFacile procède à une vérification automatisée de ma fiche d'imposition auprès des services des impôts",
"custom-text": "Afin d'améliorer votre dossier, veuillez expliquer ci-dessous pourquoi vous ne recevez pas d'avis d'impositon. Votre explication sera ajoutée à votre dossier :",
"files": "Documents",
"register": "Register",
"field-required": "This field is required"
},
"fr": {
"my-name": "Vous avez un avis d’imposition à votre nom",
"my-parents": "Vous êtes rattaché fiscalement à vos parents",
"less-than-year": "Vous êtes en France depuis moins d’un an",
"other-tax": "Autre",
"accept-verification": "J'accepte que DossierFacile procède à une vérification automatisée de ma fiche d'imposition auprès des services des impôts",
"custom-text": "Afin d'améliorer votre dossier, veuillez expliquer ci-dessous pourquoi vous ne recevez pas d'avis d'impositon. Votre explication sera ajoutée à votre dossier :",
"files": "Documents",
"register": "Enregistrer",
"field-required": "Ce champ est requis"
}
}
</i18n>
