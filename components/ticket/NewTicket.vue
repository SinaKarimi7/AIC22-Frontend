<template>
  <div>

    <v-form ref="form" class="pa-2" v-model="valid" lazy-validation>
      <div class="title-input mb-3">عنوان</div>
      <v-text-field
          v-model="ticket.title"
          :counter="30"
          :rules="[v => !!v || 'عنوان نمی تواند خالی باشد!', v => (v && v.length <= 30) || 'عنوان نمی تواند بیشتر از 30 کاراکتر باشد!']"
          label=""
          placeholder="عنوان تیکیت"
          style="border-radius: 15px"
          required
          outlined
      ></v-text-field>

      <div class="title-input  mb-3">متن</div>

      <Editor @update="updateText"/>

      <h5 class="mt-4">تگ مربوط به تیکت خود را انتخاب کنید!</h5>
      <div>
        <v-chip-group column active-class="primary" class=""  v-model="ticket.tag">
          <v-chip outlined v-for="tag in tags" :key="tag">
            {{ tag }}
          </v-chip>
        </v-chip-group>
      </div>
      <div style="display: flex; justify-content: flex-start  ;" class="mt-6 ml-2">
        <v-btn color="primary" :disabled="!valid" class="mb-2 v-btn--primary" @click="created(ticket)" width="150px " :loading="loading">
            ایجاد
        </v-btn>
      </div>
    </v-form>
  </div>
</template>

<script>
import Editor from '../editor/Editor';

export default {
  components: {Editor},
  props: ['toggleNewTicket'],
  async fetch() {
    this.loading = true;
    await this.$axios.$get('/communication/tags').then(res => {
      this.data = res.data;
      this.status_code = res.status_code;
      this.tags = [];
      this.tagsId = [];
      this.data.forEach(item => {
        this.tags.push(item.title);
        this.tagsId.push(item.id);
      });
    }).catch(reason => {
      this.$toast.error('مشکلی در لود تگ ها به وجود آمده است!');

    });
    this.loading = false;
  },
  data() {
    return {
      valid: true,
      loading: false,

      data: [],
      status_code: 200,
      ticket: {
        tag: '',
        title: '',
        text: '',
      },
      tags: [],
      tagsId: [],
    };
  },
  methods: {
    updateText(val) {
      this.ticket.text = val;
    },
    validate() {
      this.$refs.form.validate();
    },
    reset() {
      this.$refs.form.reset();
      this.$refs.form.resetValidation();
    },
    async created(answer) {
      answer.html = answer.text;
      answer.tag = this.tagsId[answer.tag];
      this.$axios.$post('communication/', answer).then(res => {
        this.$toast.success('تیکت شما ثبت شد!');
        this.$refs.form.reset();
        this.$refs.form.resetValidation();
        this.toggleNewTicket();

      }).catch(reason => {
        this.$toast.error('خطایی در ثبت تیکت به وجود آمد!');
      });
    },
  },
};
</script>

<style>
.in {
  display: flex;
  justify-content: center;
}
.title-input{
}
</style>
