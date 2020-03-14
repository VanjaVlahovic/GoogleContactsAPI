
var clientId ="790225650669-44snk6q03hinrk7ad3ucfav3vbov9h5v.apps.googleusercontent.com";
var apiKey = "OS8_N7E7OZEP0xLOTRh4fW_q";
var scopes = 'https://www.google.com/m8/feeds/';
var contacts=[];
var novKontakt;
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
        //console.log(contactEntry);
        if (typeof (contactEntry.id) != "undefined") {
            if (typeof (contactEntry.id.$t) != "undefined") {
                    contact['id'] = contactEntry.id.$t;
                
            }
        }
        if (typeof (contactEntry.gd$etag) != "undefined"){ 
                    contact['etag'] = contactEntry.gd$etag;
        }
        // Get Full Name.
        if (typeof (contactEntry.gd$name) != "undefined") {
            if (typeof (contactEntry.gd$name.gd$fullName) != "undefined") {
                if (typeof (contactEntry.gd$name.gd$fullName.$t) != "undefined") {
                    contact['name'] = contactEntry.gd$name.gd$fullName.$t;
                }
            }
        }
        
        // Get Phone Number
        if (typeof (contactEntry['gd$phoneNumber']) != "undefined") {
            var phoneNumber = contactEntry['gd$phoneNumber'];
            for (var j = 0; j < phoneNumber.length; j++) {
                if (typeof (phoneNumber[j]['$t']) != "undefined") {
                    var phone = phoneNumber[j]['$t'];
                    contact['phone'] = phone;
                }
            }
        }
        // get Email Address
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
       //document.write("https://www.google.com/m8/feeds/contacts/default/thin?alt=json&access_token="+authorizationResult.access_token+"&max-results=500&v=3.0");
        var settings = {
            "async": true,
            "crossDomain": true,
            "url": "https://www.google.com/m8/feeds/contacts/default/thin?alt=json&access_token="+authorizationResult.access_token+"&max-results=500&v=3.0",
            "method": "GET"
          }
          
          $.ajax(settings).done(function (response) {
            unesi(response);        
            izmeni(authorizationResult);                     
          }); 
        
    }
}
function izmeni(){
    var atribut=document.getElementById("atribut").value;
    var tip=document.getElementById("tipovi").value;
    var ime=document.getElementById("Imekontakta");
    var telefon=document.getElementById("TelefonKontakta");
    var email=document.getElementById("EmailKontakta");
    var upozorenje=document.getElementById("upozorenje");
   
    var kontakt;
    
    if (tip=="Ime"){
        for(var i=0; i<contacts.length; i++){
            if(contacts[i]['name']==atribut){
                kontakt=contacts[i];
            }
        }
    }
    if(tip=="Telefon"){
        for(var i=0; i<contacts.length; i++){
            if(contacts[i]['phone']==atribut){
                contact=contact[i];
            }
        }
    }
    if (tip=="Email"){
        for(var i=0; i<contacts.length; i++){
            if(contacts[i]['email']==atribut){
                contact=contact[i];
            }
        }
    }
    
    ime.innerHTML="Ime: "+kontakt['name'];
    telefon.innerHTML="Telefon: "+kontakt['phone'];
    email.innerHTML="E-mail: "+kontakt['email'];
    upozorenje.innerHTML="Molimo Vas unesite sve podatke iako neki ostaju isti :)";
    
    novKontakt=kontakt;
    
   
}  
function put(){
    var imeNovo=document.getElementById("ime");
    var telefonNov=document.getElementById("telefon");
    var emailNov=document.getElementById("email");

    alert("Etag="+novKontakt['etag']+"   Id="+ novKontakt['id']);
    var id=novKontakt['id'];
    var ostatak=id.substring(66);
    var etag=novKontakt['etag'];

    var obj= {
        "id":{"$t":"http://www.google.com/m8/feeds/contacts/default/"+ostatak},
        "gd$etag":etag,
        "category": [{
        "scheme": "http://schemas.google.com/g/2005#kind",
        "term": "http://schemas.google.com/contact/2008#contact"
                     }],
        "gd$name": {"gd$fullName": {"$t": imeNovo}},
        "gd$email":[{"address":emailNov, "primary":"true","rel":"http://schemas.google.com/g/2005#other"}],
        "gd$phoneNumber": [{
        "rel": "http://schemas.google.com/g/2005#mobile",
        "uri": telefonNov,
        "$t": telefonNov
        }]
    }


    var settings = {
        "async": true,
        "crossDomain": true,
        "url": "https://www.google.com/m8/feeds/contacts/default/"+ostatak+"/thin?alt=json&access_token="+rez.access_token+"&max-results=500&v=3.0",
        "method": "PUT",
        "headers": {
          "If-Match": etag,
          "Content-Type": "application/json",
        },
        "processData": false,
        "data": obj
    }
    $.ajax(settings).done(function (response) {
        alert("OK");
    });
}
