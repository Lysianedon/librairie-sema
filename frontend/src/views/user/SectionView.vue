<template>
    <div class="sectionview">
        <div class="nav">
          <NavbarUser/>
        </div>
        <div class="sidebar-mobile">
        <SidebarMobile/>
        </div>

        <a href="" id="scrollToTop"></a>
      <!--------------- BANNER ---------------->
      <div class="login-wrapper" v-if="!isUserConnected">
          <img :src="getImgUrl(getCurrentBanner)" alt="banniere" srcset="" class="banner home-banner">

          <div class="login-invitation">
            <h2>Qui va là ?
                <font-awesome-icon icon="fa-solid fa-eye" />
                <font-awesome-icon icon="fa-solid fa-eye" />
            </h2>
            <h2>Vous devez être connecté pour accéder à cette page </h2>
            <router-link to="/login" class="router-link">
                <h2 class="login-link">Connectez-vous ou inscrivez vous ici</h2>
            </router-link>
          </div>
      </div>
      <div class="content" v-if="isUserConnected">
        <!--------------- BANNER ---------------->
        <img
        :src="getImgUrl(getCurrentBanner)"
        alt="banniere"
        srcset=""
        :class="`banner ${getCurrentCollection === 'favoris'? 'banner-favorites' : ''} ${getCurrentCollection === 'ma bibliotheque'? 'banner-library' : ''} ${getCurrentCollection !== 'favoris' && getCurrentCollection !== 'ma bibliotheque' ?  'home-banner' : ''}`">

        <!------------- SECTION TITLE -------------->
        <h2 class="is-size-2 has-text-centered-mobile"> {{getCurrentSectionTitle}} ({{getNumberOfBooks}})</h2>
        <!------------- BOOK SECTION ------------------>
        <div class="books" v-if="getCurrentBookCollection">
          <div v-for="(book, index) in getCurrentBookCollection" :key="index">
            <Book
            class="book-component"
            v-if="isUserConnected"
            :bookSelection="[book]"
            :currentCollection="getCurrentCollection"
            :fromGeneralCollection="fromGeneralCollection"
            @updated-library="updateLibrary"/>
          </div>
        </div>
        <!------------- SECTION TITLE FOR "ALREADY READ" SECTION-------------->
        <h2 class="is-size-2 has-text-centered-mobile"
        v-if="getCurrentCollection === currentCollections.bibliotheque">Déjà lus ({{getNumberOfAlreadyReadBooks}})</h2>
        <h3 class="h3-notif-no-books-read" v-if="getAlreadyReadBookCollection.length === 0 && getCurrentCollection === currentCollections.bibliotheque">Vous n'avez lu aucun livre pour le moment.</h3>
        <!------------- BOOK SECTION : ALREADY READ ------------------>
        <div class="container-books"
        v-if="getCurrentCollection === currentCollections.bibliotheque">
            <Book
            :bookSelection="getAlreadyReadBookCollection"
            :fromGeneralCollection="false"
            :fromAlreadyReadCollection="true"
            :currentCollection="'alreadyread'"
            @updated-library="updateLibrary"/>
        </div>

      <a href="#scrollToTop" class="haut-de-page">
        <b-tooltip
                label="Remonter"
                type="is-white"
                position="is-top">
            <font-awesome-icon icon="fa-solid fa-arrow-up" />
        </b-tooltip>
      </a>

      <div class="footer-component">
          <FooterComponent/>
      </div>
    </div>
      </div>
</template>

<script>
import axios from 'axios'
import NavbarUser from '@/components/NavbarUser.vue'
import FooterComponent from '@/components/FooterComponent.vue'
import SidebarMobile from '@/components/SidebarMobile.vue'
import Book from '@/components/Book.vue'
export default {
  name: 'SectionView',
  data () {
    return {
      isUserConnected: false,
      banners: {
        default: 'banner.png',
        favoris: 'banner-favorites.png',
        bibliotheque: 'banner-library.png'
      },
      sectionTitles: {
        bibliotheque: 'Ma bibliothèque',
        favoris: 'Mes favoris',
        contes: 'Tous les contes',
        romans: 'Tous les romans',
        nouveautes: 'Toutes les nouveautés',
        biographies: 'Toutes les biographies',
        bandesDessinees: 'Toutes les bandes dessinées',
        tousLesLivres: 'Tous les livres'
      },
      currentCollections: {
        bibliotheque: 'ma bibliotheque',
        favoris: 'favoris',
        contes: 'Tous les contes',
        romans: 'Tous les romans',
        nouveautes: 'Toutes les nouveautés',
        biographies: 'Toutes les biographies',
        bandesDessinees: 'Toutes les bandes dessinées',
        tousLesLivres: 'Tous les livres'
      },
      backendRouteNames: {
        userLibrary: 'user/library',
        books: 'books'
      },
      bookSelection: [],
      alreadyReadBookSelection: [],
      fromGeneralCollection: false
    }
  },
  components: {
    NavbarUser,
    FooterComponent,
    SidebarMobile,
    Book
  },
  computed: {
    getCurrentSectionTitle (sectionview) {
      sectionview = this.$route.params.sectionview
      if (sectionview === 'favoris') {
        return this.sectionTitles.favoris
      } else if (sectionview === 'ma-bibliotheque') {
        return this.sectionTitles.bibliotheque
      } else if (sectionview === 'contes') {
        return this.sectionTitles.contes
      } else if (sectionview === 'romans') {
        return this.sectionTitles.romans
      } else if (sectionview === 'nouveautes') {
        return this.sectionTitles.nouveautes
      } else if (sectionview === 'biographies') {
        return this.sectionTitles.biographies
      } else if (sectionview === 'tous-les-livres') {
        return this.sectionTitles.tousLesLivres
      } else if (sectionview === 'bandes-dessinees') {
        return this.sectionTitles.bandesDessinees
      }
      return 'Tous les résultats'
    },
    getCurrentCollection (sectionview) {
      sectionview = this.$route.params.sectionview
      if (sectionview === 'favoris') {
        return this.currentCollections.favoris
      } else if (sectionview === 'ma-bibliotheque') {
        return this.currentCollections.bibliotheque
      } else if (sectionview === 'contes') {
        return this.currentCollections.contes
      } else if (sectionview === 'romans') {
        return this.currentCollections.romans
      } else if (sectionview === 'nouveautes') {
        return this.currentCollections.nouveautes
      } else if (sectionview === 'biographies') {
        return this.currentCollections.biographies
      } else if (sectionview === 'tous-les-livres') {
        return this.currentCollections.tousLesLivres
      } else if (sectionview === 'bandes-dessinees') {
        return this.currentCollections.bandesDessinees
      }
      return 'not found'
    },
    getCurrentBanner (sectionview) {
      sectionview = this.$route.params.sectionview
      if (sectionview === 'favoris') {
        return this.banners.favoris
      } else if (sectionview === 'ma-bibliotheque') {
        return this.banners.bibliotheque
      }
      return this.banners.default
    },
    getBackendRoute (sectionview) {
      sectionview = this.$route.params.sectionview
      if (sectionview === 'favoris' || sectionview === 'ma-bibliotheque') {
        return this.backendRouteNames.userLibrary
      }
      return this.backendRouteNames.books
    },
    getCurrentBookCollection (sectionview) {
      sectionview = this.$route.params.sectionview
      return this.getBookCollection(sectionview, this.getBackendRoute)
    },
    getAlreadyReadBookCollection () {
      return this.alreadyReadBookSelection
    },
    getNumberOfBooks () {
      return this.bookSelection.length
    },
    getNumberOfAlreadyReadBooks () {
      return this.alreadyReadBookSelection.length
    }
  },
  mounted () {
    // CHECKING IF USER IS CONNECTED:
    axios
      .get(`http://localhost:${process.env.VUE_APP_PORT}/user/`, { withCredentials: true })
      .then(res => {
        if (res.data.success) {
          this.isUserConnected = true
        } else {
          this.isUserConnected = false
        }
      })
      .catch(err => {
        this.isUserConnected = false
        return console.error(err)
      })
    const params = this.$route.params.sectionview
    // IF THE ROUTE IS REDIRECTING TO THE USER'S LIBRARY, WE GET THE INFOS AND SET THE PARAMETERS ACCORDINGLY (SECTION TITLE, BACKEND'S ROUTE URL, ETC):
    if (params === 'ma-bibliotheque') {
      this.fromGeneralCollection = false

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            this.bookSelection = res.data.userLibrary.allBooks
            this.alreadyReadBookSelection = res.data.userLibrary.alreadyRead
          }
        })
        .catch(err => {
          return err
        })
    }
    // IF THE ROUTE IS REDIRECTING TO THE USER'S FAVORITES:
    if (params === 'favoris') {
      this.fromGeneralCollection = false

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            this.bookSelection = res.data.userLibrary.favorites
          }
        })
        .catch(err => {
          return err
        })
    }
    // IF THE ROUTE IS REDIRECTING TO THE TALES:
    if (params === 'contes') {
      this.fromGeneralCollection = true

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            let bookSelection = res.data.bookList
            bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'conte')
            this.bookSelection = bookSelection
          }
        })
        .catch(err => err)
    }
    // IF THE ROUTE IS REDIRECTING TO THE NOVELS:
    if (params === 'romans') {
      this.fromGeneralCollection = true

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            let bookSelection = res.data.bookList
            bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'roman')
            this.bookSelection = bookSelection
          }
        })
        .catch(err => err)
    }
    // IF THE ROUTE IS REDIRECTING TO THE COMICS:
    if (params === 'bandes-dessinees') {
      this.fromGeneralCollection = true

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            let bookSelection = res.data.bookList
            bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'bande dessinée')
            this.bookSelection = bookSelection
          }
        })
        .catch(err => err)
    }
    // IF THE ROUTE IS REDIRECTING TO THE BIOGRAPHIES:
    if (params === 'biographies') {
      this.fromGeneralCollection = true

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            let bookSelection = res.data.bookList
            bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'biographie' || book.genre.toLowerCase() === 'autobiographie')
            this.bookSelection = bookSelection
          }
        })
        .catch(err => err)
    }
    // IF THE ROUTE IS REDIRECTING TO SEMA'S WHOLE BOOK LIST:
    if (params === 'tous-les-livres') {
      this.fromGeneralCollection = true

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            console.log(res.data)
            this.bookSelection = res.data.bookList
          }
        })
        .catch(err => err)
    }
    // IF THE ROUTE IS REDIRECTING TO THE NEWLY ADDED BOOKS:
    if (params === 'nouveautes') {
      this.fromGeneralCollection = true

      // Getting the book selection:
      axios
        .get(`http://localhost:${process.env.VUE_APP_PORT}/${this.backendRouteName}`, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            console.log(res.data)
            this.bookSelection = res.data.bookList
            const bookSelection = res.data.bookList
            // Step 1 : Flatten the obj before sorting them:
            const flattenObj = (ob) => {
            // The object which contains the final result
              const result = {}
              // loop through the object "ob"
              for (const i in ob) {
                // We check the type of the i using typeof() function and recursively call the function again
                if ((typeof ob[i]) === 'object' && !Array.isArray(ob[i])) {
                  const temp = flattenObj(ob[i])
                  for (const j in temp) {
                  // Store temp in result
                    result[i + j] = temp[j]
                  }
                } else { // Else store ob[i] in result directly
                  result[i] = ob[i]
                }
              }
              return result
            }
            const flattenBooksOjects = bookSelection.map(book => {
              book = flattenObj(book)
              return book
            })
            // Step 2 : sorting the books:
            const newlyAddedBooks = flattenBooksOjects.slice().sort((a, b) => b.dateAddedparsedFormat - a.dateAddedparsedFormat)
            // console.log('newlyAddedBooks', newlyAddedBooks)
            this.bookSelection = newlyAddedBooks
          }
        })
        .catch(err => err)
    }
  },
  methods: {
    getImgUrl (pic) {
      return require('@/assets/' + pic)
    },
    getBookCollection (sectionview, backendRoute) {
      // IF THE ROUTE IS REDIRECTING TO THE USER'S LIBRARY, WE GET THE INFOS AND SET THE PARAMETERS ACCORDINGLY (SECTION TITLE, BACKEND'S ROUTE URL, ETC):
      if (sectionview === 'ma-bibliotheque') {
        this.currentCollection = 'ma bibliotheque'
        this.fromGeneralCollection = false

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              this.bookSelection = res.data.userLibrary.allBooks
              this.alreadyReadBookSelection = res.data.userLibrary.alreadyRead
              return this.bookSelection
            }
          })
          .catch(err => {
            return err
          })
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO THE USER'S FAVORITES:
      if (sectionview === 'favoris') {
        this.currentCollection = 'favoris'
        this.fromGeneralCollection = false

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              this.bookSelection = res.data.userLibrary.favorites
              return this.bookSelection
            }
          })
          .catch(err => {
            return err
          })
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO THE TALES:
      if (sectionview === 'contes') {
        this.currentCollection = 'contes'
        this.fromGeneralCollection = true

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              let bookSelection = res.data.bookList
              bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'conte')
              this.bookSelection = bookSelection
              return this.bookSelection
            }
          })
          .catch(err => err)
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO THE NOVELS:
      if (sectionview === 'romans') {
        this.currentCollection = 'romans'
        this.fromGeneralCollection = true

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              let bookSelection = res.data.bookList
              bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'roman')
              this.bookSelection = bookSelection
              return this.bookSelection
            }
          })
          .catch(err => err)
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO THE COMICS:
      if (sectionview === 'bandes-dessinees') {
        this.backendRouteName = 'books'
        this.currentCollection = 'bandes dessinées'
        this.fromGeneralCollection = true

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              let bookSelection = res.data.bookList
              bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'bande dessinée')
              this.bookSelection = bookSelection
              return this.bookSelection
            }
          })
          .catch(err => err)
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO THE BIOGRAPHIES:
      if (sectionview === 'biographies') {
        this.currentCollection = 'biographies'
        this.fromGeneralCollection = true

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              let bookSelection = res.data.bookList
              bookSelection = bookSelection.filter(book => book.genre.toLowerCase() === 'biographie' || book.genre.toLowerCase() === 'autobiographie')
              this.bookSelection = bookSelection
              return this.bookSelection
            }
          })
          .catch(err => err)
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO SEMA'S WHOLE BOOK LIST:
      if (sectionview === 'tous-les-livres') {
        this.currentCollection = 'tous les livres'
        this.fromGeneralCollection = true

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
            //   console.log(res.data)
              this.bookSelection = res.data.bookList
              return this.bookSelection
            }
          })
          .catch(err => err)
        return this.bookSelection
      }
      // IF THE ROUTE IS REDIRECTING TO THE NEWLY ADDED BOOKS:
      if (sectionview === 'nouveautes') {
        this.currentCollection = 'nouveautes'
        this.fromGeneralCollection = true

        // Getting the book selection:
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/${backendRoute}`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
            //   console.log(res.data)
              this.bookSelection = res.data.bookList
              const bookSelection = res.data.bookList
              // Step 1 : Flatten the obj before sorting them:
              const flattenObj = (ob) => {
                // The object which contains the final result
                const result = {}
                // loop through the object "ob"
                for (const i in ob) {
                // We check the type of the i using typeof() function and recursively call the function again
                  if ((typeof ob[i]) === 'object' && !Array.isArray(ob[i])) {
                    const temp = flattenObj(ob[i])
                    for (const j in temp) {
                      // Store temp in result
                      result[i + j] = temp[j]
                    }
                  } else { // Else store ob[i] in result directly
                    result[i] = ob[i]
                  }
                }
                return result
              }
              const flattenBooksOjects = bookSelection.map(book => {
                book = flattenObj(book)
                return book
              })
              // Step 2 : sorting the books:
              const newlyAddedBooks = flattenBooksOjects.slice().sort((a, b) => b.dateAddedparsedFormat - a.dateAddedparsedFormat)
              //  console.log('newlyAddedBooks', newlyAddedBooks)
              this.bookSelection = newlyAddedBooks
              return this.bookSelection
            }
          })
          .catch(err => err)
        return this.bookSelection
      }
    },
    addToLibrary (bookId) {
      // Adding the book to the corresponding collection : favorites or library:
      const bookToAddID = bookId
      axios
        .post(`http://localhost:${process.env.VUE_APP_PORT}/user/library/allbooks`, { bookToAddID }, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            // console.log(res.data)
            // Displaying a success notification
            this.$buefy.toast.open({
              message: 'Ajouté à la bibliothèque',
              type: 'is-success'
            })
          }
        })
        .catch(err => {
          // Displaying an error notification
          this.$buefy.toast.open({
            message: 'Livre déjà dans votre bibliothèque',
            type: 'is-danger'
          })
          return console.error(err)
        })
    },
    addToFavorites (bookId) {
      const bookToAddID = bookId
      // Adding the book to the general library first, and then to the favorites:
      this.addToLibrary(bookId)
      axios
        .post(`http://localhost:${process.env.VUE_APP_PORT}/user/library/favorites`, { bookToAddID }, { withCredentials: true })
        .then(res => {
          if (res.data.success) {
            // console.log(res.data)
            // Displaying a success notification
            this.$buefy.toast.open({
              message: 'Ajouté aux favoris',
              type: 'is-success'
            })
          }
        })
        .catch(err => {
          // Displaying an error notification
          this.$buefy.toast.open({
            message: 'Livre déjà dans vos favoris',
            type: 'is-danger'
          })
          return console.log(err)
        })
    },
    deleteFromCollection (bookId, currentCollection) {
      if (currentCollection === 'ma bibliotheque') {
        const bookToDeleteID = bookId
        axios
          .delete(`http://localhost:${process.env.VUE_APP_PORT}/user/library/allbooks`, { withCredentials: true, data: { bookToDeleteID } })
          .then(res => {
            if (res.data.success) {
            //   console.log(res.data)
              // Displaying a success notification
              this.$buefy.toast.open({
                message: 'Supprimé de votre bibliothèque',
                type: 'is-success'
              })
            }
          })
        // Deleting the book from the favorites too, if found in the collection:
        axios
          .delete(`http://localhost:${process.env.VUE_APP_PORT}/user/library/favorites`, { withCredentials: true, data: { bookToDeleteID } })
          .catch(err => {
            // Displaying an error notification
            this.$buefy.toast.open({
              message: 'Oups...Erreur, veuillez réessayer',
              type: 'is-danger'
            })
            return console.log(err)
          })
      }
      // --------------------------- IF COLLECTION = FAVORITES ---------------
      if (currentCollection === 'favoris') {
        const bookToDeleteID = bookId
        axios
          .delete(`http://localhost:${process.env.VUE_APP_PORT}/user/library/favorites`, { withCredentials: true, data: { bookToDeleteID } })
          .then(res => {
            if (res.data.success) {
            //   console.log(res.data)
              // Displaying a success notification
              this.$buefy.toast.open({
                message: 'Supprimé de vos favoris',
                type: 'is-success'
              })
            }
          })
          .catch(err => {
            // Displaying an error notification
            this.$buefy.toast.open({
              message: 'Oups...Erreur, veuillez réessayer',
              type: 'is-danger'
            })
            return console.log(err)
          })
      }
    },
    updateLibrary (payload) {
      if (payload.updatedAlreadyRead) {
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/user/library`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              this.alreadyReadBookSelection = res.data.userLibrary.alreadyRead
            }
            return null
          })
          .catch(err => {
            return err
          })
      }
      if (payload.updatedLibrary) {
        axios
          .get(`http://localhost:${process.env.VUE_APP_PORT}/user/library`, { withCredentials: true })
          .then(res => {
            if (res.data.success) {
              this.bookSelection = res.data.userLibrary
            }
            return null
          })
          .catch(err => {
            return err
          })
      }
    }
  }
}
</script>

<style scoped>
.content, .login-wrapper{
  margin: 3% auto 0 20vw;
  width: 92vw;
}

.nav{
  position: fixed;
  top: 6%;
}

.sidebar-mobile{
  display: none;
}

.banner{
  width: 82%;
  margin: auto;
  height: 42vh;
  border: 2px solid rgb(88, 88, 88);
  border-radius: 15px;
}

.login-invitation{
    cursor: pointer;
    /* border: 1px solid black; */
    border-radius: 9px;
    box-shadow: 0 1px 7px rgba(0,0,0,0.12), 0 1px 4px rgba(0,0,0,0.24);
    width: 60%;
    margin: 10vh auto;
    margin-left: 10%;
    /* height: 0vh; */
    display: flex;
    flex-direction:column;
    align-items:center;
    justify-content: center;
    background-color: #FFF1CC;
}
.login-invitation h2{
  font-size: 1.6rem;
  line-height: 300%;
}
.login-invitation h2:nth-child(1){
  font-size: 1.7rem;
}

.login-link{
  cursor: pointer;
}
.login-link:hover{
  font-weight: bold;
}
.books{
  display: flex;
  flex-wrap: wrap;
  width: 92%;
}

.book-component{
  margin: 0 1.2vw;
  width: 17vw;
  height: 70vh;
}

.h3-notif-no-books-read{
  margin-left: 1vw;
  color: rgb(171, 168, 168);
  padding-top: 1vh;
  font-size: 1.4rem;
}

h2{
  font-family: 'Ibarra Real Nova', serif;
  color: black;
  cursor: default;
}

h3{
    font-size: 1.1rem ;
    font-weight: 600 !important;
    line-height: 155% !important;
}

p{
  line-height: 190%;
  width: 90%;
  text-align: justify;
}

button{
  font-weight: bold;
}
.haut-de-page{
  margin-left: 72vw;
  padding-bottom: .8%;
}
.footer-component{
    margin-top: 3%;
}

/* RESPONSIVE --  RESPONSIVE -- RESPONSIVE -- RESPONSIVE -- RESPONSIVE -- */

@media(max-width: 1170px){
  .nav{
    display: none;
  }
  .login-wrapper{
    margin: 3% auto 4% 5vw;
  }
  .login-invitation{
    width: 90%;
    padding: 3%;
    text-align: center;
    margin: 6% auto auto auto;
  }
  .sidebar-mobile{
    display: block;
    position: fixed;
    top: 3%;
    left: 2%;
    z-index: 2;
  }
  .content{
    margin: auto;
    width: 95vw;
  }

  .banner{
    width: 95%;
    margin: 3% 2% !important;
    height: 42vh;
  }
  .books{
    margin: auto;
  }
  .book-component{
    width: 29vw;
    margin: 0;
  }

  .haut-de-page{
    padding:2%;
    margin-left: 88vw;
  }

  .footer-component{
    width: 100vw;
  }
}

/*-------------  TABLET MODE ------------ */

@media(max-width: 630px){
  .book-component{
    height: fit-content;
    width: 40vw;
    margin: 3%;
  }
}

@media(max-width: 500px){
  .books{
    flex-direction: column;
    width: 100vw;
    overflow-x: hidden;
  }
}
/*-------------  MOBILE MODE ------------ */
@media(max-width: 450px){

  .login-invitation h2{
    font-size: 1.6rem !important;
    line-height: 200%;
  }
  .container-books{
    width: 50%;
    padding: 0;
  }
  .banner-library{
    content:url('@/assets/banner-library-mobile.png') !important;
  }
  .banner-favorites{
    content:url('@/assets/banner-favorites-mobile.png') !important;
  }
  .home-banner{
    content:url('@/assets/home-banner-mobile.png') !important;
  }

  .book-component{
    height: fit-content;
    width: 80vw;
    margin:auto;
  }

  h2{
    font-size: 1.9rem;
  }

  .haut-de-page{
    padding:6%;
     margin-left: 75vw;
  }
}
@media(max-width: 330px){
  .login-invitation h2{
    font-size: 1.3rem !important;
  }
  .book-component{
    height: fit-content;
    width: 78vw;
    margin-left:0;
  }
}

</style>
