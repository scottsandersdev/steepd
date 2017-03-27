<template>
  <div id="app" class="container">
    <a class="user-profile"
      @click="signOut">
      <img class="user-profile__img" v-bind:src="user.photoURL">
    </a>
    <div class="page-header">
      <h1></h1>
    </div>

    <div class="panel panel-default">
      <div class="panel-heading">
        <h3>Add Tea</h3>
      </div>
      <div class="panel-body">
        <form id="form" class="form-inline" v-on:submit.prevent="addTea">
          <div class="form-group">
            <label for="teaName">Name:</label>
            <input type="text" id="teaName" class="form-control" v-model="newTea.name">
          </div>
          <div class="form-group">
            <label for="teaType">Type:</label>
            <input type="text" id="teaType" class="form-control" v-model="newTea.type">
          </div>
          <input type="submit" class="btn btn-primary" value="Add Tea">
        </form>
      </div>
    </div>

    <div class="panel panel-default">
      <div class="panel-heading">
        <h3>Teas</h3>
      </div>
      <div class="panel-body">
        <table class="table table-striped">
          <thead>
            <tr>
              <th>
                Name
              </th>
              <th>
                Type
              </th>
              <th>
                Delete
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="tea in teasFiltered">
              <td>{{tea.name}}</td>
              <td>{{tea.type}}</td>
              <td><span class="btn-trash glyphicon glyphicon-trash" @click.prevent="removeTea(tea)"></span></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>

import firebase from 'firebase'

let config = {
  apiKey: "AIzaSyDsaWFL55FnNnq9J18LtIxRG-bRuBU47Es",
  authDomain: "steepd-d58dd.firebaseapp.com",
  databaseURL: "https://steepd-d58dd.firebaseio.com",
  storageBucket: "steepd-d58dd.appspot.com",
  messagingSenderId: "447567076359"
};

let app = firebase.initializeApp(config);
let db = app.database();

let teasRef = db.ref('teas');
let methods = {};
let computed = {};

methods.addTea = function() {
  teasRef.push(this.newTea);
  this.newTea.name = '';
  this.newTea.type = '';
};

methods.removeTea = function(tea) {
  teasRef.child(tea['.key']).remove();
};

/**
 * handles signin to google
 * @returns {void}
 */
methods.signIn = function() {
  if (!firebase.auth().currentUser) {
    var provider = new firebase.auth.GoogleAuthProvider();
    firebase.auth().signInWithRedirect(provider);
  } else {
    this.signOut();
  }
}

methods.signOut = function() {
  firebase.auth().signOut();
}


/**
 * Checks if user signed in on load
 * @returns {void}
 */
const initApp = function() {
  this.$nextTick(() => {
    firebase.auth().getRedirectResult().then((result) => {
      if (result.credential) {
        this.user = result.user;
      }
    });

    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.user = user;
      } else {
        this.signIn();
      }
    });
  });
}

computed.teasFiltered = function() {
  return this.teas.sort(function (a, b) {
    return a.name.toLowerCase().localeCompare(b.name.toLowerCase());
  });
}

export default {
  name: 'app',
  firebase: {
    teas: teasRef
  },
  data() {
    return {
      colors: ['EDF8A1','ADBD58','332918','98816C','EEE8E2','0E0E0E','9BC3A8','A7AB5E','482723','060D10'],
      user: {},
      newTea:{
        name: '',
        type: '',
        steepingDirections: {}
      }
    }
  },
  computed: computed,
  methods: methods,
  mounted: initApp
}
</script>

<style lang="scss">
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
.btn-trash {
  cursor: pointer;
}

.user-profile {
  cursor: pointer;
  display: block;
  position: absolute;
  right: 20px;
  top: 20px;
}

.user-profile__img {
  border-radius: 50%;
  width: 80px;
}

</style>
