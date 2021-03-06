<template>
  <div class="container">
    <v-card class="elevation-12">
      <v-toolbar flat dark color="#07889B">
        <v-toolbar-title class="tableTitle">Edit contact</v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
      <form class="formContainer">
        <v-text-field
          v-validate="'required|max:20'"
          v-model="contact.firstName"
          :counter="20"
          :error-messages="errors.collect('name')"
          label="Firstname"
          data-vv-name="name"
          prepend-icon="person"
          color="#07889B"
          required>
        </v-text-field>
        <v-text-field
          v-validate="'required|max:20'"
          v-model="contact.lastName"
          :counter="20"
          :error-messages="errors.collect('name')"
          label="Lastname"
          data-vv-name="name"
          color="#07889B"
          required>
        </v-text-field>
        <v-text-field
          v-validate="'required|email'"
          v-model="contact.email"
          :error-messages="errors.collect('email')"
          label="E-mail"
          data-vv-name="email"
          color="#07889B"
          prepend-icon="fa fa-envelope"
          required>
        </v-text-field>
        <div class="numbersContainer">
          <v-tooltip bottom>
            <v-btn
              slot="activator"
              @click="addNumber"
              class="addBtn"
              dark
              small
              fab
              color="#07889B"
              outline>
              <v-icon dark>add</v-icon>
            </v-btn>
            <span>Add number</span>
          </v-tooltip>
        </div>
        <v-layout v-for="(number, index) in contact.PhoneNumbers" :key="index" row wrap>
          <v-text-field
            v-validate="'required|numeric'"
            v-model="number.number"
            :error-messages="errors.collect('numeric')"
            class="numberField"
            label="Phone number"
            data-vv-name="numeric"
            color="#07889B"
            mask="phone"
            prepend-icon="fa fa-phone"
            required>
          </v-text-field>
          <v-select
            v-validate="'required'"
            :items="types"
            v-model="number.numberType"
            :error-messages="errors.collect('select')"
            class="typeField"
            label="Type"
            data-vv-name="select"
            color="#07889B"
            required>
          </v-select>
          <v-checkbox
            v-validate="'required'"
            v-model="number.isMain"
            class="isMain"
            label="Set this number as primary?"
            data-vv-name="checkbox"
            type="checkbox"
            color="#07889B"
            required>
          </v-checkbox>
          <v-tooltip right class="delBtn">
            <v-icon v-if="index>=0" slot="activator" @click="deleteNumber(index)" color="error"> delete </v-icon>
            <span>Delete number</span>
          </v-tooltip>
        </v-layout>
        <v-btn @click="submit" color="#07889B" small flat outline>submit</v-btn>
        <v-btn @click="dialog = true" small flat outline>cancel</v-btn>
        <v-dialog v-model="dialog" max-width="290">
          <v-card>
            <v-card-text> Are you sure you want to cancel? </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn @click="dialog = false" color="#07889B" flat> No </v-btn>
              <v-btn @click="cancel" color="#07889B" flat> Yes </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-snackbar v-model="snackbar" color="error">
          There must be at least and only one primary number
          <v-btn @click="snackbar=false" dark flat >Close</v-btn>
        </v-snackbar>
      </form>
    </v-card>
  </div>
</template>

<script>
import contactService from '@/services/contactService';

export default {
  props: {
    id: { default: 0, type: [ String, Number ] }
  },
  data() {
    return {
      contact: {
        firstName: '',
        lastName: '',
        email: '',
        PhoneNumbers: [ { number: '', numberType: null, isMain: true } ]
      },
      types: [ 'Fax', 'Private', 'Business' ],
      dialog: false,
      snackbar: false,
      counter: 0,
      dictionary: {
        attributes: { email: 'E-mail Address', numeric: 'Phone number' },
        custom: {
          name: {
            required: () => 'Name fields can not be empty',
            max: 'Name fields may not be greater than 10 characters'
          },
          select: {
            required: 'Phone number type is required'
          }
        }
      }
    };
  },
  methods: {
    submit() {
      this.$validator.validateAll()
        .then(result => {
          if (result) {
            this.contact.PhoneNumbers.forEach(contact => {
              if (contact.isMain === true) this.counter++;
            });
            if (this.counter === 1) {
              contactService.EditContact(this.contact)
                .then(() => this.$router.push({ name: 'contacts' }))
                .catch(err => console.log(err));
            } else {
              this.counter = 0;
              this.snackbar = true;
            }
          }
        });
    },
    cancel() {
      this.$validator.reset();
      this.$router.push({ name: 'contacts' });
    },
    addNumber() {
      this.contact.PhoneNumbers.push({ number: '', numberType: null, isMain: false });
    },
    deleteNumber(index) {
      this.contact.PhoneNumbers.splice(index, 1);
    }
  },
  mounted() {
    this.$validator.localize('en', this.dictionary);
  },
  created() {
    contactService.GetContact(this.id)
      .then(res => Object.assign(this.contact, res.data))
      .catch(err => console.log(err));
  }
};
</script>

<style scoped>
.container {
  max-width: 60%;
  margin: auto;
}
.formContainer {
  margin: 30px;
  padding-bottom: 30px;
}
.numbersContainer {
  float: right;
}
.typeField {
  max-width: 150px;
  margin-left: 20px;
}
.numberField {
  max-width: 400px;
}
.addBtn {
  margin-left: 20px;
}
.delBtn {
  padding: 18px;
}
.isMain {
  margin-left: 30px;
  max-width: 310px;
}
</style>
