<template>
  <div>
    <NakedCard v-if="user.lastName || hasDocument()">
      <template v-slot:content>
        <div v-if="user.lastName">
          <a href="#" class="rf-link">
            {{ $t("title") }}
          </a>
          <hr />
          <div class="rf-card__desc">
            <section>
              <div class="row" v-if="user.lastName">
                <div>
                  <div class="subtitle">Prénom et Nom</div>
                  {{ user.firstName }} {{ user.lastName }}
                </div>
                <div class="edit-step-btn" @click="setStep(0)">
                  <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                </div>
              </div>
              <div class="row" v-if="user.applicationType">
                <div>
                  <div class="subtitle">Type de location</div>
                  {{ $t(user.applicationType) }}
                </div>
                <div class="edit-step-btn" @click="setStep(1)">
                  <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                </div>
              </div>
            </section>
          </div>
        </div>
        <div v-if="hasDocument()">
          <a href="#" class="rf-link">
            {{ $t("second-title") }}
          </a>
          <hr />
          <div class="rf-card__desc">
            <section>
              <div class="row" v-if="hasDoc('IDENTIFICATION')">
                <div class="subtitle">Pièce d’identité</div>
                <div class="row">
                  <div class="edit-step-btn" @click="openDoc('IDENTIFICATION')">
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setTenantStep(1)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="hasDoc('RESIDENCY')">
                <div class="subtitle">Justificatif de domicile</div>
                <div class="row">
                  <div class="edit-step-btn" @click="openDoc('RESIDENCY')">
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setTenantStep(2)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="hasDoc('PROFESSIONAL')">
                <div class="subtitle">
                  Justificatif de situation professionelle
                </div>
                <div class="row">
                  <div class="edit-step-btn" @click="openDoc('PROFESSIONAL')">
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setTenantStep(3)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="hasDoc('FINANCIAL')">
                <div class="subtitle">Justificatif de revenu</div>
                <div class="row">
                  <div class="edit-step-btn" @click="openDoc('FINANCIAL')">
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setTenantStep(4)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="hasDoc('TAX')">
                <div class="subtitle">Avis d’imposition</div>
                <div class="row">
                  <div class="edit-step-btn" @click="openDoc('TAX')">
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setTenantStep(5)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
            </section>
          </div>
        </div>
        <div v-if="hasGuarantor()">
          <a href="#" class="rf-link">
            {{ $t("third-title") }}
          </a>
          <hr />
          <div class="rf-card__desc">
            <section>
              <div class="row" v-if="guarantorHasDoc('IDENTIFICATION')">
                <div class="subtitle">Pièce d’identité</div>
                <div class="row">
                  <div
                    class="edit-step-btn"
                    @click="openGuarantorDoc('IDENTIFICATION')"
                  >
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setGuarantorStep(1)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="guarantorHasDoc('RESIDENCY')">
                <div class="subtitle">Justificatif de domicile</div>
                <div class="row">
                  <div
                    class="edit-step-btn"
                    @click="openGuarantorDoc('RESIDENCY')"
                  >
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setGuarantorStep(2)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="guarantorHasDoc('PROFESSIONAL')">
                <div class="subtitle">
                  Justificatif de situation professionelle
                </div>
                <div class="row">
                  <div
                    class="edit-step-btn"
                    @click="openGuarantorDoc('PROFESSIONAL')"
                  >
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setGuarantorStep(3)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="guarantorHasDoc('FINANCIAL')">
                <div class="subtitle">Justificatif de revenu</div>
                <div class="row">
                  <div
                    class="edit-step-btn"
                    @click="openGuarantorDoc('FINANCIAL')"
                  >
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setGuarantorStep(4)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
              <div class="row" v-if="guarantorHasDoc('TAX')">
                <div class="subtitle">Avis d’imposition</div>
                <div class="row">
                  <div class="edit-step-btn" @click="openGuarantorDoc('TAX')">
                    <i class="icon color--primary rf-p-1w icon-Eye-2"></i>
                  </div>
                  <div class="edit-step-btn" @click="setGuarantorStep(5)">
                    <i class="icon color--primary rf-p-1w icon-Pen-4"></i>
                  </div>
                </div>
              </div>
            </section>
          </div>
        </div>
      </template>
    </NakedCard>
    <Modal v-show="isDocModalVisible" @close="isDocModalVisible = false">
      <template v-slot:body>
        <div class="rf-container">
          <div class="row justify-content-center">
            <div class="col-12 col-md-8">
              <div v-for="f in files" v-bind:key="f.id">
                <img :src="getUrl(f.path)" v-if="isImage(f.path)">
                <PdfViewer :src="getUrl(f.path)" v-if="!isImage(f.path)"></PdfViewer>
              </div>
            </div>
          </div>
        </div>
      </template>
    </Modal>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { mapState } from "vuex";
import NakedCard from "df-shared/src/components/NakedCard.vue";
import { User } from "df-shared/src/models/User";
import { Guarantor } from "df-shared/src/models/Guarantor";
import { DfFile } from "df-shared/src/models/DfFile";
import Modal from "df-shared/src/components/Modal.vue";
import PdfViewer from "../components/PdfViewer.vue";

@Component({
  components: { NakedCard, Modal, PdfViewer },
  computed: {
    ...mapState({
      user: "user",
      tenantStep: "tenantStep",
      selectedGuarantor: "selectedGuarantor",
    }),
  },
})
export default class EditSummary extends Vue {
  user!: User;
  selectedGuarantor!: Guarantor;
  isDocModalVisible = false;
  files: DfFile[] = [];

  setTenantStep(n: number) {
    this.$store.commit("setTenantSubstep", n);
    this.setStep(2);
  }

  setGuarantorStep(n: number) {
    this.$store.commit("setGuarantorSubstep", n);
    this.setStep(3);
  }

  setStep(n: number) {
    this.$store.commit("setStep", n);
  }
  hasDocument() {
    return this.user.documents !== undefined && this.user.documents?.length > 0;
  }
  hasDoc(docType: string) {
    return this.user.documents?.find((d) => {
      return d.documentCategory === docType;
    });
  }
  guarantorHasDoc(docType: string) {
    return this.selectedGuarantor.documents?.find((d) => {
      return d.documentCategory === docType;
    });
  }
  hasGuarantor() {
    return (
      this.selectedGuarantor?.documents !== undefined &&
      this.selectedGuarantor.documents.length > 0
    );
  }

  openDoc(documentCategory: string) {
    const docs = this.user.documents?.filter((d) => {
      return d.documentCategory === documentCategory;
    });
    this.files = [];
    if (docs === undefined) {
      return
    }
    for (let i=0; i< docs.length ; i++) {
        this.files = this.files.concat(docs[i].files || []);
    }
    this.isDocModalVisible = true;
  }

  openGuarantorDoc(documentCategory: string) {
    const docs = this.selectedGuarantor.documents?.filter((d) => {
      return d.documentCategory === documentCategory;
    });
    this.files = [];
    if (docs === undefined) {
      return
    }
    for (let i=0; i< docs.length ; i++) {
        this.files = this.files.concat(docs[i].files || []);
    }
    this.isDocModalVisible = true;
  }

  isImage(path: string) {
    return !path.endsWith('pdf');
  }

  getUrl(path: string) {
    return `//${process.env.VUE_APP_API_URL}/api/file/tenants_file/${path}`;
  }
}
</script>

<style scoped lang="scss">
.row {
  display: flex;
  justify-content: space-between;
  padding: 0.2rem;
  div {
    align-self: center;
  }
}
.edit-step-btn {
  align-self: center;
  padding: 0.2rem;
  cursor: pointer;
}
.subtitle {
  font-weight: bold;
}
</style>

<i18n>
{
"en": {
"title": "Information du locataire",
"second-title": "Documents du locataire",
"third-title": "Documents du garant",
"ALONE": "Seul",
"COUPLE": "En couple",
"GROUP": "En colocation"
},
"fr": {
"title": "Information du locataire",
"second-title": "Documents du locataire",
"third-title": "Documents du garant",
"ALONE": "Seul",
"COUPLE": "En couple",
"GROUP": "En colocation"
}
}
</i18n>
