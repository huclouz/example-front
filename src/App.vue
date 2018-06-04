<template>
  <v-app id="inspire">
    <v-navigation-drawer
      :clipped="$vuetify.breakpoint.lgAndUp"
      v-model="drawer"
      fixed
      app>

      <v-list class="pt-0" dense>
        <v-list-tile @click.native="memberDialog=true">
          <v-list-tile-action>
            <v-icon>account_circle</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title>회원가입</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>


      <v-list class="pt-0" dense v-if="!token">
        <v-list-tile @click.native="loginDialog=true">
          <v-list-tile-action>
            <v-icon>face</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title>로그인</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>

      <v-list class="pt-0" dense>
        <v-list-tile v-for="item in bookmarks" :key="item.bookTitle" @click="detail(item.bookIsbn, item.bookTitle, item.id)">
          <v-list-tile-action>
            <v-icon>bookmark</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title>{{ item.bookTitle }}</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>

    </v-navigation-drawer>
    <v-toolbar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      color="blue darken-3"
      dark
      app
      fixed
    >
      <v-toolbar-title style="width: 300px" class="ml-0 pl-3">
        <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
        <span class="hidden-sm-and-down">BookSearch</span>
      </v-toolbar-title>

      <v-flex xs1>
        <v-select
          l1
          flat
          :items="bookcat"
          item-text="text"
          item-value="value"
          v-model="cat"
          class="hidden-sm-and-down"
          label="범주"/>
      </v-flex>
      &nbsp;&nbsp;

      <v-flex xs1>
        <v-select
          l1
          flat
          :items="pageSize"
          item-text="text"
          item-value="value"
          v-model="selectedPagesize"
          class="hidden-sm-and-down"
          label="개수"/>
      </v-flex>
      &nbsp;&nbsp;

      <v-flex xs1 v-if="token">
        <v-select
          l1
          flat
          :items="histories"
          item-text="searchKeyword"
          item-value="searchKeyword"
          v-model="searchText"
          class="hidden-sm-and-down"
          label="검색이력"/>
      </v-flex>

      &nbsp;&nbsp;
      <v-text-field
        flat
        solo-inverted
        v-model="searchText"
        prepend-icon="search"
        label="검색어를 입력 후 검색버튼을 눌러주세요"
        class="hidden-sm-and-down"></v-text-field>
        <v-btn color="success" @click="search(0)" :loading="dataloading">검색</v-btn>

      <v-spacer></v-spacer>
    </v-toolbar>

    <v-content>
      <div>
        <v-btn color="info" v-if="!isEnd" @click="search(1)" :loading="dataloading">{{currentPage-1}}/{{totalPage}} 더보기</v-btn>
      </div>
      <v-layout style="margin-top:30px; margin-bottom:30px" v-for="item in searchData" :key="item.isbn">
        <v-flex xs10 sm8 offset-sm3>
          <v-card>
            <v-card-title>
              <div>
                <h1><span class="grey--text">{{item.title}}</span></h1>
                <span>저자 : {{item.authors.toString()}}</span><br>
                <div><span>출판사 : {{item.publisher}}</span></div>
                <span>내용 : {{item.contents}}</span>
              </div>
            </v-card-title>
            <v-card-actions>
              <v-btn flat color="orange" @click="detail(item.isbn, item.title)">상세</v-btn>
              <v-btn flat color="red" target="_blank" v-bind:href="item.url">구매</v-btn>
            </v-card-actions>
          </v-card>
        </v-flex>
      </v-layout>
      <div>
        <v-btn color="info" v-if="!isEnd" @click="search(1)" :loading="dataloading">{{currentPage-1}}/{{totalPage}} 더보기</v-btn>
      </div>
    </v-content>

    <v-snackbar
      :timeout="1000"
      :multi-line="true"
      :vertical="true"
      v-model="snackbar"
    >
      {{ snackbarText }}
      <v-btn dark flat @click.native="snackbar = false">닫기</v-btn>
    </v-snackbar>

    <v-layout row justify-center>
      <v-dialog v-model="detailDialog" persistent max-width="500px">
        <v-card>
          <v-card-media v-bind:src="detailData.thumbnail" height="300">
          </v-card-media>
          <v-card-title primary-title>
            <div>
              <h1><span class="grey--text">{{detailData.title}}</span></h1>
              <div><span>저자 : {{detailData.authors}}</span></div>
              <div><span>출판사 : {{detailData.publisher}}</span></div>
              <div><span>판매여부 : {{detailData.status}}</span></div>
              <div><span>도서판매가 : <s>{{detailData.price}}원</s> -> {{detailData.sale_price}}원</span></div>
              <div>{{detailData.contents}}</div>
            </div>
          </v-card-title>
          <v-card-actions>
            <v-btn flat color="black" @click.native="detailDialog = false">닫기</v-btn>
            <v-btn flat color="red" target="_blank" v-bind:href="detailData.url">구매</v-btn>
            <v-btn v-if="!detailData.isbookmarked" flat color="blue" @click="addBookmark(detailData.isbn, detailData.title)">북마크</v-btn>
            <v-btn v-else flat color="blue" @click="deleteBookMark(detailData.bookmarkId)">북마크제거</v-btn>
            </v-card-actions>
          </v-card>
      </v-dialog>
    </v-layout>


    <v-layout row justify-center>
      <v-dialog v-model="memberDialog" persistent max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">회원가입</span>
          </v-card-title>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field label="성함" v-model="member.name" maxlength="5" counter="5" required></v-text-field>
                </v-flex>
                </v-flex>
                <v-flex xs12>
                  <v-text-field label="Email" v-model="member.email" maxlength="100" counter="100" required></v-text-field>
                </v-flex>
                <v-flex xs12>
                  <v-text-field label="비밀번호" v-model="member.password" type="password" maxlength="10" counter="10" required></v-text-field>
                </v-flex>
                <v-flex xs12>
                  <v-text-field label="비밀번호 확인" v-model="member.c_password" type="password" maxlength="10" counter="10" required></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>
            <small>*indicates required field</small>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click.native="memberDialog = false">닫기</v-btn>
            <v-btn color="blue darken-1" flat @click="register">회원가입</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-layout>

    <v-layout row justify-center>
      <v-dialog v-model="loginDialog" persistent max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">로그인</span>
          </v-card-title>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                </v-flex>
                <v-flex xs12>
                  <v-text-field label="Email" v-model="email" maxlength="100" counter="100" required></v-text-field>
                </v-flex>
                <v-flex xs12>
                  <v-text-field label="비밀번호" v-model="password" type="password" maxlength="10" counter="10" required></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="login">로그인</v-btn>
            <v-btn color="red darken-1" flat @click.native="loginDialog = false">닫기</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-layout>

  </v-app>
</template>

<script>
  import axios from 'axios'
  const crypto = require('crypto')
  const prefix = "http://localhost:8080"
  export default {
    data: () => ({
      loginDialog : false,
      dataloading : false,
      email : '',
      password : '',
      currentPage : 1,
      totalCount : '',
      token : '',
      totalPage : -1,
      snackbarText : '',
      snackbar : false,
      memberDialog : false,
      detailDialog : false,
      isEnd : true,
      dialog: false,
      drawer: null,
      cat: 'all',
      selectedPagesize: 10,
      searchText: '',
      pageSize: [
        {text: '10개씩 보기', value: 10},
        {text: '30개씩 보기', value: 30},
        {text: '50개씩 보기', value: 50},
      ],
      bookcat: [
        {text: '전체', value: 'all'},
        {text: '책제목', value: 'title'},
        {text: '저자', value: 'author'},
        {text: 'isbn', value: 'isbn'},
        {text: '출판사', value: 'publisher'}
      ],
      histories: [],
      member : {},
      bookmarks: [],
      searchData : [],
      detailData : []
    }),
    methods: {
      register : function(){
        const member = this.member
        if ( member.email == '' ){
          this.snackbarText = '이메일은 필수 입력 사항입니다.'
          this.snackbar = true
          return
        }
        if ( member.password != member.c_password ){
          this.snackbarText = '비밀번호가 일치 하지 않습니다. 확인 후 다시 입력해주세요'
          this.snackbar = true
          return
        }
        member.password = this.encrypt(member.password)
        member.c_password = this.encrypt(member.c_password)
        axios.post(prefix+'/members/signup', this.convertFormData(member))
          .then(response =>{
            this.memberDialog = false
            this.snackbarText = '축하합니다! 회원가입 되셨습니다.'
            this.snackbar = true
          }).catch(error=>{
            if (error.response.status == 409 ) {
              this.snackbarText = '이미 가입한 회원 정보입니다. 다른 이메일을 입력해주세요.'
            } else {
              this.snackbarText = '회원 가입중 오류가 생겼습니다.'
            }
            this.snackbar = true
        })
      },
      login: function () {

        if ( this.email == '' ){
          this.snackbarText = '이메일은 필수 입력 사항입니다.'
          this.snackbar = true
          return
        }

        var password = this.encrypt(this.password)
        axios({
          url : prefix+'/members/login',
          data : {
            email : this.email,
            password : password
          },
          headers:{"Content-Type":"application/json;charset=utf-8"},
          method : 'post',
          responseType : 'json'
        }).then(response =>{
          if (response.headers.authorization) {
            this.token = response.headers.authorization
            axios.defaults.headers.common['Authorization'] = this.token
            localStorage.setItem("Authorization", this.token)
            this.getHistories()
            this.getBookmarks()
          }
          this.loginDialog = false
        }).catch(error=>{
          this.snackbarText = '아이디 또는 비밀번호가 맞지 않습니다.'
          this.snackbar = true
        })

      },
      search: function (isNext) {

        this.dataloading = true
        if ( this.searchText == '') {
          this.snackbarText = '검색어를 입력해주세요'
          this.snackbar = true
          this.dataloading = false
          return
        }

        // 더보기를 통하여 들어온 경우를 뜻한다.
        if ( isNext != 1) {
          this.currentPage = 1
          this.totalCount = 0
          this.totalPage = 0
          this.isEnd = true
          this.dataloading = false
        }

        axios.get(prefix+'/books/cat/' + this.cat + '/keyword/' + encodeURI(this.searchText) + '/size/' + this.selectedPagesize + '/page/' + this.currentPage)
          .then(response => {
            if ( isNext == 1) {
              for ( var item of response.data.data) {
                this.searchData.push(item)
              }
            } else {
              this.searchData = []
              this.searchData = response.data.data
            }
            this.currentPage++
            this.totalCount = response.data.totalCount
            this.totalPage = Math.ceil(this.totalCount/this.selectedPagesize)
            this.isEnd = response.data.isEnd
            this.dataloading = false
            if (this.token != '') {
              this.getHistories()
            }
          })
          .catch(response => {
            this.snackbarText = '도서정보를 검색 중 오류가 발생하였습니다.'
            this.snackbar = true
            this.dataloading = false
          })
      },
      detail: function (isbn, title, bookmarkId) {
        var firstIsbn;
        for ( var isbn of isbn.split(' ')) {
          if ( isbn != '') {
            firstIsbn = isbn;
            break;
          }
        }
        if ( !firstIsbn ) {
          this.snackbarText = 'ISBN정보가 누락되어 도서정보를 상세하게 표시할 수 없습니다.'
          this.snackbar = true
          return;
        }
        this.detailData = {}
        axios.get(prefix+'/books/'+firstIsbn)
          .then(response => {
            this.detailDialog = true
            for (var item of response.data) {
              // isbn + title로 무결성 보장, api는 모두 or 조건이기때문에 이렇게 사용합니다.
              if ( title == item.title ) {
                item.authors = item.authors.toString()
                item.isbookmarked = this.isBookMark(isbn)
                item.bookmarkId = bookmarkId
                this.detailData = item
              }
            }
          })
          .catch(response => {
            this.snackbarText = '도서정보를 검색 중 오류가 발생하였습니다.'
            this.snackbar = true
          })
      },
      getBookmarks: function(){
        axios.get(prefix+'/books/marks')
          .then(response => {
            this.bookmarks = response.data
          })
          .catch(response => {
            this.snackbarText = '북마크 정보 조회중 오류가 발생하였습니다.'
            this.snackbar = true
          })
      },
      getHistories : function(){
        axios.get(prefix+'/books/histories')
          .then(response => {
            this.histories = response.data
          })
          .catch(response => {
            this.snackbarText = '북마크 정보 조회중 오류가 발생하였습니다.'
            this.snackbar = true
          })
      },
      addBookmark: function(isbn, title){
        if ( this.token == '' ){
          this.snackbarText = '로그인을 하셔야 사용이 가능한 기능입니다.'
          this.snackbar = true
          return
        }

        if ( this.isBookMark(isbn) ) {
          this.snackbarText = '이미 추가된 북마크입니다'
          this.snackbar = true
          return
        }
        var formData = new FormData()
        formData.append('bookIsbn', isbn)
        formData.append('bookTitle', title)
        axios.post(prefix+'/books/marks', formData)
          .then(response =>{
            this.snackbarText = '북마크에 추가되었습니다.'
            this.snackbar = true
            this.getBookmarks()
        }).catch(error=>{
          this.snackbarText = '북마크 생성 중 오류가 발생하였습니다.'
          this.snackbar = true
        })
      },
      isBookMark: function(isbn){
        return this.bookmarks.find(x => x.bookIsbn === isbn) ? true : false
      },
      convertFormData :function(object) {
        const formData = new FormData()
        Object.keys(object).forEach(key => formData.append(key, object[key]))
        return formData
      },
      deleteBookMark: function (bookmarkid) {
        axios.delete(prefix+'/books/marks/'+bookmarkid)
          .then(response =>{
            this.detailDialog = false
            this.snackbarText = '북마크가 제거 되었습니다.'
            this.snackbar = true
            this.getBookmarks()
          }).catch(error=>{
          this.snackbarText = '북마크 제거 중 오류가 발생하였습니다.'
          this.snackbar = true
        })
      }, encrypt : function(text){
        return crypto.createHash('sha512').update(text).digest('base64')
      }
    },
    created : function(){
      const token = localStorage.getItem("Authorization");
      if ( token ) {
        this.token = token
        axios.defaults.headers.common['Authorization'] = this.token
        this.getHistories()
        this.getBookmarks()
      }
    },
  }
</script>
