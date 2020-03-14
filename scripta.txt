
var clientId ="790225650669-44snk6q03hinrk7ad3ucfav3vbov9h5v.apps.googleusercontent.com";
var apiKey = "OS8_N7E7OZEP0xLOTRh4fW_q";
var scopes = 'https://www.google.com/m8/feeds/';
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
        //console.log(contactEntry);
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
        var settings = {
            "async": true,
            "crossDomain": true,
            "url": "https://www.google.com/m8/feeds/contacts/default/thin?alt=json&access_token="+authorizationResult.access_token+"&max-results=500&v=3.0",
            "method": "GET"
          }
          
          $.ajax(settings).done(function (response) {
            unesi(response);  
            prikazi("tableP");   
                                
          });
    }
} 
