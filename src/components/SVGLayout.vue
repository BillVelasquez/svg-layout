<template>
    <div ref="svgobject" >
    <object alt="Layout" :data="location" type="image/svg+xml">
			Browser not compatible
		</object>    
  </div>
</template>

<script>
export default {
  name: 'SVGLayout',
  model: {
    event: 'select'
  },  
  props: {
    location : String,   // URL of the SVG table layout file,
    prefix : String,     // Prefix to be prepended to every ID in the SVG, but not present in IDs of the status map
    idfield : String,    // JSON field used to identify the table
    status : Array,
    status_map : Object
  },

  data(){
    return   {   
    }
  },
  
  beforeMount : function() {
      // this.loadMap();
    },

  methods : {
/*    updateStatus( status_object ) {},   // Updates all tables with a full status JSON object for all tables
    updateTable( table_id, table_status, order_status ) {}  // Updates the status of only one Table, given its ID */

    update : function (){
      this.buildMap( this.$refs["svgobject"]  );
    },

// get the embedded document, if possible
    getContentDocument : function (embeddingElement) {
      if (embeddingElement.contentDocument !== null) {
        return embeddingElement.contentDocument;
      }
      try {
        var svg = embeddingElement.getSVGDocument();
      return svg;
      } catch (e) {
      console.log(e);
      }
      return null;
    },

    // eslint-disable-next-line
    select : function ( e, o ){
      // console.log( "Selecting" );
      // console.log(e);
      let selection = e.target;

      // Find the parent with clickable class
      while( !selection.classList.contains("clickable") && selection.parentNode !== null )
        selection = selection.parentNode;

      if ( selection.id == "" || selection.id == "."  || selection.parentNode == null) 
        // Invalid selection
        return;

      this.$emit( "select", selection )
    },

    getStatus : function (id) {
        if( !(this.status)) return;
        
        // Find status for ID
        let noprefix = "";
        if( !(this.prefix) || this.prefix == "" )
          noprefix = id ;
        else 
          noprefix = id.substr( this.prefix.length );
        
        let status = this.status[ noprefix ];

        if( !status ){
          // Not found as collection, try as array with id property
          status = this.status.find( k => k[this.idfield].toString() == noprefix.toString() );
        }

        return status;

      },  

    // main logic
    buildMap : function (m) {
      let children = m.children;
      let d = this.getContentDocument(children[0]),
      p = d.querySelectorAll('g.clickable');

      for(var i = 0; i < p.length; i++){
        let path = p[i];
        path.addEventListener('click', this.select );

        console.log("Event listening " + path.id);

        let status = this.getStatus( path.id );

      // Get style used for that status
        let style = (status) ? this.status_map[ status.table_status ] : null;
        
        if( style  ){
          // console.log("setting style to " + style)
          this.changeStyle( path, style);
        } 
      }
    },

    changeStyle( path, style ){
      if( !path ) return;
    //  if( path.style != null )
    try{
        path.style.fill = style;
    } catch(e){
      console.log(e);
    }
      
      let children = path.children;

      for(let child in children){
          this.changeStyle(children[child], style);
      }

    }


  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
