<template>
    <div class="container-fluid">
		<div class="row">
			<div class="col-lg-4 col-sm-3 col-xs-2"></div>
			<div class="col-lg-4 col-sm-6 col-xs-8">
				<div class="jumbotron" style="margin-top:70px;border:1px solid #4682b4;max-width:550px;min-width:370px;">	
				  <form v-on:submit.prevent="login">
					<h2 class="text-secondary text-center font-weight-bold mb-3">
					  EXPENSE TRACKER
					</h2>
					<span v-show="loginerror" class="text-danger" style="float:right">
					  <b>Login Error!! </b>
					</span>
					<div class="input-group mb-3">
					  <div class="input-group-prepend">
						<div class="input-group-text">
						  <span class="fas fa-user"></span>
						</div>
					  </div>            
					  <input type="text" class="form-control" v-model="input.username" id="username" placeholder="Username" autofocus required>
					</div>
					<div class="input-group mb-3">
					  <div class="input-group-prepend">
						<div class="input-group-text">
						  <span class="fas fa-lock"></span>
						</div>
					  </div>
					  <input type="password" class="form-control" v-model="input.password" id="password" placeholder="Password" required>
					</div>
					<button class="form-control btn btn-primary font-bold" v-on:click.prevent="login()">Login</button>
				  </form>
				  <hr>
				  <button class="form-control btn btn-danger font-bold" v-on:click="googleLogin()">Login with Google 
					<span class="fa fa-plane-paper"></span>
				  </button>
				  <hr>
				  <p class="text-warning">Don't have an account? <router-link to="/signup" class="font-italic font-weight-bold">Sign Up </router-link></p>
				</div>
			</div>
			<div class="col-lg-4 col-sm-3 col-xs-2"></div>
      <div class="spinner-border text-warning" :class="{'d-none': !showloader}" style="position:fixed;left:50%;top:30%;">      
      </div>
		</div>
	</div>
</template>

<script>
//import Vue from "vue";
import firebase from "firebase";

export default {
  data() {
    return {
      loginerror: false,
      showloader: false,
      input: {
        username: "",
        password: ""
      }
    };
  },
  beforeCreate() {
    if (this.$session.exists()) {
      this.$session.destroy();
      this.$parent.authenticated = false;
    }
  },
  methods: {
    login() {
      this.showloader = true;
      if (this.input.username != "" && this.input.password != "") {
        if (
          this.input.username == this.$parent.mockAccount.username &&
          this.input.password == this.$parent.mockAccount.password
        ) {
          var userdata = { 
              user: {
                email: "demo@et.com",
                uid: "XeIApcO56eZtCls49BK6KN8bbZB3",
                displayName: "Demo User"
              }
            }
          this.$session.start();
          this.$session.set("authUserData", userdata);
          this.$parent.authenticated = true;
          this.$store.dispatch(
            "fetchFirestoreData",
            userdata
          );
          this.showloader = false;
          this.$router.push("/dashboard");
          this.loginerror = false;
        } else {
          //console.log("The username and/or password is incorrect");
          //this.loginerror = true;
          this.emailLogin(this.input.username, this.input.password);
        }
      } else {
        alert("A username and password must be present");
        this.loginerror = true;
        this.showloader = false;
      }
    },
    emailLogin(email, password) {      
      firebase
        .auth()
        .signInWithEmailAndPassword(email, password)
        .then(result => {
          //console.log("EmailAuth:\n", result)
          var authUserData = JSON.parse(JSON.stringify(result));
          authUserData.user.displayName = email;
          this.$session.start();                           
          this.$session.set("authUserData", authUserData);     
          this.$parent.authenticated = true;
          this.$store.dispatch(
            "fetchFirestoreData",
            this.$session.get("authUserData") //.user.uid
          );
          this.$router.push("/dashboard");          
        })
        .catch(error => {
          // Handle Errors here.
          var errorCode = error.code;
          var errorMessage = error.message;
          alert(errorCode, ":", errorMessage);
          this.loginerror = true;
          this.showloader = false;
        });
    },
    googleLogin() {
      //console.log("Working on Google login feature")
      this.showloader = true;
      var provider = new firebase.auth.GoogleAuthProvider();
      firebase
        .auth()
        .signInWithPopup(provider)
        .then(result => {
          // This gives you a Google Access Token. You can use it to access the Google API.
          var token = result.credential.accessToken;
          // The signed-in user info.
          var user = result.user;
          // ...
          //this.$store.dispatch("setAuthUserData", result);
          console.log("Google login success:\n", JSON.stringify(result, 3, 3));
          this.$session.start();
          this.$session.set("authUserData", result);
          this.$parent.authenticated = true;
          this.$store.dispatch(
            "fetchFirestoreData",
            this.$session.get("authUserData") //.user.uid
          );
          this.$router.push("/dashboard");
          this.loginerror = false;
        })
        .catch(error => {
          // Handle Errors here.
          var errorCode = error.code;
          var errorMessage = error.message;
          this.loginerror = true;
          this.showloader = false;
          alert("Error!!\n" + errorMessage);
        });
    }
  }
};
</script>
