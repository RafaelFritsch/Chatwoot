<template>
  <modal :show.sync="show" :on-close="onClose">
    <div class="column content-box">
      <woot-modal-header :header-title="pageTitle" />
      <form class="row medium-8" @submit.prevent="editCannedResponse()">
        <div class="medium-12 columns">
          <label :class="{ error: $v.shortCode.$error }">
            {{ $t('CANNED_MGMT.EDIT.FORM.SHORT_CODE.LABEL') }}
            <input
              v-model.trim="shortCode"
              type="text"
              :placeholder="$t('CANNED_MGMT.EDIT.FORM.SHORT_CODE.PLACEHOLDER')"
              @input="$v.shortCode.$touch"
            />
          </label>
        </div>

        <div class="medium-12 columns">
          <label :class="{ error: $v.content.$error }">
            {{ $t('CANNED_MGMT.EDIT.FORM.CONTENT.LABEL') }}
          </label>
          <div class="editor-wrap">
            <woot-message-editor
              v-model="content"
              class="message-editor"
              :class="{ editor_warning: $v.content.$error }"
              :enable-variables="true"
              :enable-canned-responses="false"
              :placeholder="$t('CANNED_MGMT.EDIT.FORM.CONTENT.PLACEHOLDER')"
              @blur="$v.content.$touch"
            />
          </div>
        </div>
        <div class="modal-footer">
          <div class="medium-12 columns">
            <woot-submit-button
              :disabled="
                $v.content.$invalid ||
                  $v.shortCode.$invalid ||
                  editCanned.showLoading
              "
              :button-text="$t('CANNED_MGMT.EDIT.FORM.SUBMIT')"
              :loading="editCanned.showLoading"
            />
            <button class="button clear" @click.prevent="onClose">
              {{ $t('CANNED_MGMT.EDIT.CANCEL_BUTTON_TEXT') }}
            </button>
          </div>
        </div>
      </form>
    </div>
  </modal>
</template>

<script>
/* eslint no-console: 0 */
import { required, minLength } from 'vuelidate/lib/validators';
import WootMessageEditor from 'dashboard/components/widgets/WootWriter/Editor';
import WootSubmitButton from '../../../../components/buttons/FormSubmitButton';
import alertMixin from 'shared/mixins/alertMixin';
import Modal from '../../../../components/Modal';

export default {
  components: {
    WootSubmitButton,
    Modal,
    WootMessageEditor,
  },
  mixins: [alertMixin],
  props: {
    id: { type: Number, default: null },
    edcontent: { type: String, default: '' },
    edshortCode: { type: String, default: '' },
    onClose: { type: Function, default: () => {} },
  },
  data() {
    return {
      editCanned: {
        showAlert: false,
        showLoading: false,
      },
      shortCode: this.edshortCode,
      content: this.edcontent,
      show: true,
    };
  },
  validations: {
    shortCode: {
      required,
      minLength: minLength(2),
    },
    content: {
      required,
    },
  },
  computed: {
    pageTitle() {
      return `${this.$t('CANNED_MGMT.EDIT.TITLE')} - ${this.edshortCode}`;
    },
  },
  methods: {
    setPageName({ name }) {
      this.$v.content.$touch();
      this.content = name;
    },
    resetForm() {
      this.shortCode = '';
      this.content = '';
      this.$v.shortCode.$reset();
      this.$v.content.$reset();
    },
    editCannedResponse() {
      // Show loading on button
      this.editCanned.showLoading = true;
      // Make API Calls
      this.$store
        .dispatch('updateCannedResponse', {
          id: this.id,
          short_code: this.shortCode,
          content: this.content,
        })
        .then(() => {
          // Reset Form, Show success message
          this.editCanned.showLoading = false;
          this.showAlert(this.$t('CANNED_MGMT.EDIT.API.SUCCESS_MESSAGE'));
          this.resetForm();
          setTimeout(() => {
            this.onClose();
          }, 10);
        })
        .catch(error => {
          this.editCanned.showLoading = false;
          const errorMessage =
            error?.message || this.$t('CANNED_MGMT.EDIT.API.ERROR_MESSAGE');
          this.showAlert(errorMessage);
        });
    },
  },
};
</script>
<style scoped lang="scss">
::v-deep {
  .ProseMirror-menubar {
    display: none;
  }

  .ProseMirror-woot-style {
    min-height: 12.5rem;

    p {
      font-size: var(--font-size-default);
    }
  }

  .message-editor {
    border: 1px solid var(--s-200);
  }
}
</style>
