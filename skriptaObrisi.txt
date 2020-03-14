
var clientId ="790225650669-44snk6q03hinrk7ad3ucfav3vbov9h5v.apps.googleusercontent.com";
var apiKey = "OS8_N7E7OZEP0xLOTRh4fW_q";
var scopes = "https://www.google.com/m8/feeds/";
var contacts=[];
function init() {
    gapi.client.setApiKey(apiKey);
    window.setTimeout(authorize);
}

function authorize() {
    gapi.auth.authorize({client_id: clientId, scope: scopes, immediate: false}, handleAuthorization);
}

function addRow(ime, telefon, email, podatak){  

    var table = document.getElementsByName(podatak)[0];
    var newRow = table.insertRow(table.rows.length/2+1);
                  
    var cel1 = newRow.insertCell(0);
    var cel2 = newRow.insertCell(1);
    var cel3 = newRow.insertCell(2);
                  
    cel1.innerHTML = ime;
    cel2.innerHTML = telefon;
    cel3.innerHTML = email;
}
function prikazi(podatak){
    for(var i=0;i<contacts.length;i++){
        addRow(contacts[i]['name'], contacts[i]['phone'],contacts[i]['email'], podatak);
    }
}
function unesi(baza){
    var entries = baza.feed.entry;
    var element=document.getElementById("korisnik");
    var element2=document.getElementById("mejl");
    element2.innerHTML=baza.feed.author[0].email.$t;
    element.innerHTML=baza.feed.author[0].name.$t;
    for (var i = 0; i < entries.length; i++) {
        var contactEntry = entries[i];
        var contact = [];
        
        if (typeof (contactEntry.id) != "undefined") {
            if (typeof (contactEntry.id.$t) != "undefined") {
                    contact['id'] = contactEntry.id.$t;
                
            }
        }
        if (typeof (contactEntry.gd$etag) != "undefined"){ 
                    contact['etag'] = contactEntry.gd$etag;
        }
        
        if (typeof (contactEntry.gd$name) != "undefined") {
            if (typeof (contactEntry.gd$name.gd$fullName) != "undefined") {
                if (typeof (contactEntry.gd$name.gd$fullName.$t) != "undefined") {
                    contact['name'] = contactEntry.gd$name.gd$fullName.$t;
                }
            }
        }
        
        
        if (typeof (contactEntry['gd$phoneNumber']) != "undefined") {
            var phoneNumber = contactEntry['gd$phoneNumber'];
            for (var j = 0; j < phoneNumber.length; j++) {
                if (typeof (phoneNumber[j]['$t']) != "undefined") {
                    var phone = phoneNumber[j]['$t'];
                    contact['phone'] = phone;
                }
            }
        }
        
        if (typeof (contactEntry['gd$email']) != "undefined") {
            var emailAddresses = contactEntry['gd$email'];
            for (var j = 0; j < emailAddresses.length; j++) {
                if (typeof (emailAddresses[j]['address']) != "undefined") {
                    var emailAddress = emailAddresses[j]['address'];
                    contact['email'] = emailAddress;
                }
            }
        }
        contacts.push(contact);
        
    } 
}
function handleAuthorization(authorizationResult) {
    if (authorizationResult && !authorizationResult.error) {
      // document.write("https://www.google.com/m8/feeds/contacts/default/thin?alt=json&access_token="+authorizationResult.access_token+"&max-results=500&v=3.0");
        var settings = {
            "async": true,
            "crossDomain": true,
            "url": "https://www.google.com/m8/feeds/contacts/default/thin?alt=json&access_token="+authorizationResult.access_token+"&max-results=500&v=3.0",
            "method": "GET"
          }
          
          $.ajax(settings).done(function (response) {
            unesi(response);        
            obrisi(authorizationResult);                     
          }); 
        
    }
}
function obrisi(rez){
    var atribut=document.getElementById("atribut").value;
    var tip=document.getElementById("tipovi").value;
    var id;
    var etag;
    if (tip=="Ime"){
        for(var i=0; i<contacts.length; i++){
            if(contacts[i]['name']==atribut){
                id=contacts[i]['id'];
                etag=contacts[i]['etag'];
            }
        }
        
    }
    if(tip=="Telefon"){
        for(var i=0; i<contacts.length; i++){
            if(contacts[i]['phone']==atribut){
                id=contacts[i]['id'];
                etag=contacts[i]['etag'];
            }
        }
    }
    if (tip=="Email"){
        for(var i=0; i<contacts.length; i++){
            if(contacts[i]['email']==atribut){
                id=contacts[i]['id'];
                etag=contacts[i]['etag'];
            }
        }
    }
    var ostatak=id.substring(66);
    alert("ID="+ostatak);
    alert("Etag="+etag);
    var settings = {
        "async": true,
        "crossDomain": true,
        "url":"https://www.google.com/m8/feeds/contacts/default/"+ostatak+"/thin?alt=json&access_token="+rez.access_token+"&max-results=500&v=3.0",
        "method": "DELETE",
        "headers": {
          "If-Match": etag,
            "cache-control": "no-cache",
            //"Postman-Token": "b9890bbf-614b-4d51-a85f-f95cadff7df8"
        }
      }
      $.ajax(settings).done(function (response) {
        console.log(response);
      });
}
