# ChangeRegistration

If the app recognizes during the app registration that the SDL HMI language (voice/TTS and/or display) does not match the app language, the app will be able (but does not need) to change this registration with changeRegistration prior to app being brought into focus.

### HMI Status Requirements ###

HMILevel any.

### Request ###

<table border="1" rules="all">
  		<tr>
  			<th>Name</th>
  			<th>Type</th>
  			<th>Description</th>
        <th> Req.</th>
  			<th>Notes</th>
  			<th>Version Available</th>
  		</tr>
  		<tr>
  			<td>Language</td>
  			<td>Language</td>
  			<td>Requested SDL voice engine (VR+TTS) language registration.</td>
        <td>Y</td>
  			<td></td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>hmiDisplayLanguage</td>
  			<td>Language</td>
  			<td>Request display language registration.</td>
        <td>Y</td>
  			<td>Minvalue=0 <br>Maxvalue=2000000000</td>
  			<td>SmartDeviceLink 2.0</td>
 		</tr>
     <tr>
  			<td>appName</td>
  			<td>String</td>
  			<td>Request new app name registration</td>
        <td>N</td>
        <td>maxlength:100</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>ttsName</td>
  			<td>TTSChunk</td>
  			<td>Request new ttsName registration</td>
        <td>N</td>
        <td>minsize:1<br> maxsize:100</td>
  			<td>SmartDeviceLink 2.0</td>
 		</tr>
      <tr>
  			<td>ngnMediaScreenAppName</td>
  			<td>String</td>
  			<td>Request new app short name registration</td>
        <td>N</td>
        <td>maxlength: 100</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>vrSynonyms</td>
  			<td>String</td>
  			<td>Request new VR synonyms registration</td>
        <td>N</td>
        <td>maxlength: 40<br>minsize:1<br>maxsize:100</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
   </table>

### Response ###

### Non-default Result Codes: ###
```
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- APPLICATION_NOT_REGISTERED
- GENERIC_ERROR
- REJECTED  
- DISALLOWED
```
### Related Operations ###
```
RegisterAppInterface
```
