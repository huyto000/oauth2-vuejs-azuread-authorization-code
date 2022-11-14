<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button v-if="!isLogin" @click="this.loginAzure">Login Azure</button>
    <h3>Hello {{ this.name }}</h3>
    <p>
      For a guide and recipes on how to configure / customize this project,<br />
      check out the
      <a href="https://cli.vuejs.org" target="_blank" rel="noopener"
        >vue-cli documentation</a
      >.
    </p>
    <h3>Installed CLI Plugins</h3>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },

  data() {
    return {
      isLogin: false,
      name: '',
    }
  },

  methods: {
    async loginAzure() {
    var state = this.generateRandomString();
    sessionStorage.setItem("pkce_state", state);

    // Create and store a new PKCE code_verifier (the plaintext random secret)
    var code_verifier = this.generateRandomString();
    sessionStorage.setItem("pkce_code_verifier", code_verifier);

    // Hash and base64-urlencode the secret to use as the challenge
    var code_challenge = await this.pkceChallengeFromVerifier(code_verifier);

    // Build the authorization URL
    var url = "xxxxxxxxxxxxxx"
        + "?response_type=code"
        + "&client_id="+encodeURIComponent("xxxxxxxxxxxxxxxx")
        + "&state="+encodeURIComponent(state)
        + "&scope="+encodeURIComponent("xxxxxxxxxxxxxxxxxxxxx")
        + "&redirect_uri="+encodeURIComponent("http://localhost:8080/")
        + "&code_challenge="+encodeURIComponent(code_challenge)
        + "&code_challenge_method=S256"
        ;
      window.location.href = url
    },

    async getToken(code) {
      let formData = new FormData();
      formData.append("client_id", "xxxxxxxxxxxxxxxxxxxxxxx");
      formData.append("scope", "xxxxxxxxxxxxxxxxxxxxxx");

      formData.append("code", code);

      formData.append("redirect_uri", "http://localhost:8080/");
      formData.append("grant_type", "authorization_code");
      formData.append(
        "code_verifier",
        sessionStorage.getItem("pkce_code_verifier")
      );
      await fetch(
        "xxxxxxxxxxxxxxxxxxxx",
        {
          body: formData,
          method: "post",
        }
      )
        .then((res) => res.json())
        .then((res) => {
          sessionStorage.setItem("token", res.access_token);
          console.log(1);
        })
        .catch((err) => console.log(err));
    },
    parseQueryString(string) {
    if(string == "") { return {}; }
    var segments = string.split("&").map(s => s.split("=") );
    var queryString = {};
    segments.forEach(s => queryString[s[0]] = s[1]);
    return queryString;
},
    // Generate a secure random string using the browser crypto functions
    generateRandomString() {
      var array = new Uint32Array(28);
      window.crypto.getRandomValues(array);
      return Array.from(array, (dec) =>
        ("0" + dec.toString(16)).substr(-2)
      ).join("");
    },

    // Calculate the SHA256 hash of the input text.
    // Returns a promise that resolves to an ArrayBuffer
    sha256(plain) {
      const encoder = new TextEncoder();
      const data = encoder.encode(plain);
      return window.crypto.subtle.digest("SHA-256", data);
    },

    // Base64-urlencodes the input string
    base64urlencode(str) {
      // Convert the ArrayBuffer to string using Uint8 array to conver to what btoa accepts.
      // btoa accepts chars only within ascii 0-255 and base64 encodes them.
      // Then convert the base64 encoded to base64url encoded
      //   (replace + with -, replace / with _, trim trailing =)
      return btoa(String.fromCharCode.apply(null, new Uint8Array(str)))
        .replace(/\+/g, "-")
        .replace(/\//g, "_")
        .replace(/=+$/, "");
    },

    // Return the base64-urlencoded sha256 hash for the PKCE challenge
    async pkceChallengeFromVerifier(v) {
      var hashed = await this.sha256(v);
      return this.base64urlencode(hashed);
    },

    async getName() {
      console.log("Authorization code: ", sessionStorage.getItem("token"))
      console.log(2)
        await fetch(
        "http://localhost:8081/foos",
        {
          method: "get",
          headers: {
          'Authorization': 'Bearer ' + sessionStorage.getItem("token"),
  }
        }
      ).catch((err) => {
          console.log(err)
          this.loginAzure()
        })
        .then((res) => res.text())
        .then((res) => {
          console.log(res);
          this.name = res
        })
        

    }
  },

  actions: {},

  async mounted() {
    const queryString = window.location.search;
    var q = this.parseQueryString(window.location.search.substring(1));

// Check if the server returned an error string
    if(q.error) {
        alert("Error returned from authorization server: "+q.error);
    }

    if(q.code) {

    // Verify state matches what we set at the beginning
    if(sessionStorage.getItem("pkce_state") != q.state) {
        alert("Invalid state");
    } else {
      const urlParams = new URLSearchParams(queryString);
      const code = urlParams.get("code");
      console.log("Code ne:", code);
      window.history.replaceState({}, null, "/");
      await this.getToken(code)
      .then(async () => {
        await this.getName()
      })
      .catch((err) => console.log(err));
  }
    sessionStorage.removeItem("pkce_state");
    sessionStorage.removeItem("pkce_code_verifier");
}
if(sessionStorage.getItem('token')) {
  this.getName()
}else {
  this.loginAzure();
}
  
},
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
