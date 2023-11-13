<template>
  <div class="columns is-centered">
    <div class="column is-two-thirds">
      <article class="dx">
        <h1 class="is-size-2-desktop is-size-3-mobile is-bold mb-5 has-text-left">
          Data Explorer Workbench
        </h1>
        <div class="columns is-centered">
          <div class="column is-one-half">
            <b-message title="Public beta notice">
                This <a href="https://en.wikipedia.org/wiki/Exploratory_data_analysis" rel="nofollow noopener noreferrer" target="_blank">EDA</a> tool is 
                in <strong>public beta</strong> and is free to test on any 
                <a href="https://github.com/MainakRepositor/Datasets" rel="nofollow noopener noreferrer" target="_blank">datasets</a>. 
                When the beta closes it will require a one time purchase to access. 
                You can find out more about the tool in 
                the <a href="https://shedloadofcode.com/store" class="">Store</a> or watch the 
                <a href="https://www.youtube.com/watch?v=fev2rpGxh-U" rel="nofollow noopener noreferrer" target="_blank">tutorial video</a>.
            </b-message>
          </div>
          <div class="column is-one-half">
            <b-message title="Privacy and data disclaimer" 
              id="privacy-message-box" 
              activearia-close-label="Close message">
              No data from your CSV is sent to 
              a server or anywhere else. All the data stays on your computer and the 
              calculations are made within your browser using JavaScript. Datasets over 20,000 rows 
              will slow down the tool considerably.
            </b-message>
          </div>
        </div>
      </article>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DataExplorerWorkbench',
  props: {
    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
body, button, input, optgroup, select, textarea {
    font-family: BlinkMacSystemFont,-apple-system,"Segoe UI",Roboto,Oxygen,Ubuntu,Cantarell,"Fira Sans","Droid Sans","Helvetica Neue",Helvetica,Arial,sans-serif;
}

article {
  margin-top: 20px;

  p {
    font-size: 20px;
  }
}

article figcaption>a, article p>a {
    color: #4a4a4a;
    text-decoration: underline;
    font-weight: 600;
}

article h1, article h2, article h3, article h4, article h5, article h6 {
  margin: 20px 0 10px;
  color: rgb(74, 74, 74);
  padding: 20px 0 0;
  font-weight: 700;
  -webkit-font-smoothing: antialiased;
  cursor: text;
  position: relative;
}

article figcaption>a:hover, article p>a:hover {
    text-decoration: none;
}

a:hover {
    color: #363636;
}

article figcaption>a:active, article figcaption>a:focus, article p>a:active, article p>a:focus {
    border-bottom: 2px solid #4a4a4a;
    text-decoration: none;
}
</style>