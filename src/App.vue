<script setup>
import SignIn from './components/SignIn.vue'
import FamousPoets from './components/FamousPoets.vue'
import { createClient } from '@supabase/supabase-js'
import { SupabaseAuthClient } from '@supabase/supabase-js/dist/module/lib/SupabaseAuthClient'
</script>

<template>
  <header>
    <router-link to="/">Go to Home</router-link>
    <img alt="Poetry" class="logo" src="./assets/logo.png" width="125" height="125" />
    <div class="wrapper" id="signOut">
      <div>
        <SignIn msg="Poet ! Tell us who you are !" />
      </div>
      <label>email: </label><br>
      <input type="email" required v-model="email" placeholder="username@domain.tld"><br>
      <label>password: </label><br>
      <input type="password" required v-model="passwd"><br>
      <button v-on:click="register()">Sign Up</button>
      <button v-on:click="login()">Sign In</button>

    </div>
    <div class="hidden" id="addPoem">
      <div>
        <SignIn msg="Write your poem !" />
      </div>
      <h3>The poem remains private, until you make it public</h3>
      <label>Poem's title</label><br>
      <input type="text" required name="title" v-model="title" placeholder="edit me"><br>
      <label>Poem's content</label><br>
      <textarea required name="content" v-model="content" placeholder="edit me" rows="10" cols="50">Your poem here ...
        </textarea> <br>

      <label>Poem's language</label><br>
      <input type="text" required name="language" v-model="language" placeholder="edit me" rows="10" cols="10" />

      <label>Illustration: </label>

      <input type="file" id="file" name="file" placeholder="my file" accept="image/png, image/jpeg"><br>
      <!--<img id="illustration" src="./assets/null.png" alt="poem illustration" width="75" height="75"/><br>-->
      <input type="checkbox" v-model="hidden" value=true />
      <label>Hidden poem</label><br>
      <label>Filter by title :</label>
      <input v-on:keyup.enter="filterpoems()" type="text" placeholder="Filter poems" v-model="text" />
      <br><button v-on:click="createPoem()">Add the poem</button>
      <button v-on:click="fetchpoems()">List of poems</button><br>
      <label for="poemtitle" id="poemtitle" style="color: teal;font-weight: 500;"> ... </label>
      <img id="poemillustration" src="./assets/null.jpg" alt="poem images" width="75" height="75"
        style="background-color:gray;" /><br>
      <label for="poemlanguage" id="poemlanguage"> ... </label><br>
      <textarea id="poemcontent" readonly rows="10" cols="50"> ... </textarea> <br>
      <button v-on:click="nextPoem()">Next poem</button><br>
      <hr />
    </div>

  </header>

  <main>
    <FamousPoets />
  </main>
</template>

<script>
const SUPABASE_URL = 'https://ufexjukbdzuneolctojt.supabase.co'
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVmZXhqdWtiZHp1bmVvbGN0b2p0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjM1OTU4NjMsImV4cCI6MTk3OTE3MTg2M30.iow6grnMT1H2Uui1rhO7uN05r3B2kau3zaP2Q5ahpOM'
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY)

var poemsList
var currentpoem

export default {
  methods: {
    //this method allows a new user to sign up the system. Once done, the user receives an email
    //asking for account validation. Once the validation made the user is added to the system
    async register() {
      try {
        const { user, session, error } = await supabase.auth.signUp({
          email: this.email,
          password: this.passwd
        })
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },
    //this method allows the already registred user to log in the system.
    //only authenticated users can later add or read the poems
    async login() {
      try {
        const { user, session, error } = await supabase.auth.signIn({
          email: this.email,
          password: this.passwd
        })
        if (error) throw error;

        else {
          document.getElementById('signOut').style.visibility = 'hidden'
          document.getElementById('addPoem').style.visibility = 'visible'
        }
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },
    //this method allows to add new poem for the authenticated user (after sign in) 
    //it is called when the user click on the add poem button after being entered
    //the title, the content, the visibility and the associated illustration
    async createPoem() {
      var res;

      const { data: objects, error } = await supabase.storage
        .from('images')
        .upload(supabase.auth.user().id + "_" + file.files[0].name, file.files[0])

      res = supabase.storage
        .from('images')
        .getPublicUrl(supabase.auth.user().id + "_" + file.files[0].name).data.publicURL;

      try {
        const { data, error } = await supabase
          .from('poems')
          .insert([
            { hidden: this.hidden, email: this.email, title: this.title, content: this.content, language: this.language, illustrationurl: res }
          ])
        if (error) throw (error)
      } catch (error) { alert(error.error_description || error.message) }
    },
    //this method allows to extract all readable poems of the authenticated user
    //including his peoms and the not hidden poems. This policy is implemented by the supabase system 
    async fetchpoems() {
      try {
        const { data, error } = await supabase
          .from('poems')
          .select()
        poemsList = data
        if (error) throw error;
        if (data.length > 0) {
          document.getElementById('poemtitle').innerHTML = data[0].title + "    "
          document.getElementById('poemcontent').value = data[0].content
          document.getElementById('poemillustration').src = data[0].illustrationurl
          document.getElementById('poemlanguage').innerHTML = data[0].language + "     "
        }
        currentpoem = 0;
      } catch (error) {
        alert(error.error_description || error.message);
      }
    },
    //this function allows to display the next accessibe poem for the current user
    //the fetch button should be selected before
    nextPoem() {
      if (currentpoem < poemsList.length - 1) {
        currentpoem++
        document.getElementById('poemtitle').innerHTML = poemsList[currentpoem].title + "    "
        document.getElementById('poemcontent').value = poemsList[currentpoem].content
        document.getElementById('poemillustration').src = poemsList[currentpoem].illustrationurl
        document.getElementById('poemlanguage').innerHTML = poemsList[currentpoem].language + "    "
      } else currentpoem = 0
      document.getElementById('poemtitle').innerHTML = poemsList[currentpoem].title + "    "
      document.getElementById('poemcontent').value = poemsList[currentpoem].content
      document.getElementById('poemillustration').src = poemsList[currentpoem].illustrationurl
      document.getElementById('poemlanguage').innerHTML = poemsList[currentpoem].language + "    "
    },

    async filterpoems() {
      try {
        const { data, error } = await supabase
          .from('poems')
          .select()
          .like('title', "%" + this.text + "%")
        poemsList = data
        if (error) throw error;
        if (data.length > 0) {
          document.getElementById('poemtitle').innerHTML = data[0].title + "   "
          document.getElementById('poemcontent').value = data[0].content
          document.getElementById('poemillustration').src = data[0].illustrationurl
          document.getElementById('poemlanguage').innerHTML = data[0].language + "     "
        }
        currentpoem = 0;
      } catch (error) {
        alert(error.error_description || error.message)
      }
    }
  }
}
</script>

<style>
@import './assets/base.css';

header .hidden {
  visibility: hidden;
  overflow: hidden;
  display: flex;
  display: inline-block;
  place-items: flex-start;
  flex-wrap: wrap;
}

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

@media (hover: hover) {
  a:hover {
    background-color: hsla(160, 100%, 37%, 0.2);
  }
}

@media (min-width: 1024px) {
  body {
    display: flex;
    place-items: center;
  }

  #app {
    display: grid;
    grid-template-columns: 1fr 1fr;
    padding: 0 2rem;
  }

  header {
    display: inline-block;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    display: inline-block;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }
}
</style>