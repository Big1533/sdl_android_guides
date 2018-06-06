# PutFile

Used to push a binary data onto the SDL module from a mobile device, such as icons and album art.

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
  			<td>FileName</td>
  			<td>String</td>
  			<td>File reference name.</td>
                  <td>Y</td>
  			<td>Maxlength=500</td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>fileType</td>
  			<td>FileType</td>
  			<td>Selected file type.</td>
                 <td>Y</td>
  			<td></td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>persistentFile</td>
  			<td>Boolean</td>
  			<td>Indicates if the file is meant to persist between sessions / ignition cycles.<br> If set to TRUE,then the system will aim to persist this file through session / cycles.<br>While files with this designation will have priority over others,<br> they are subject to deletion by the system at any time.<br> In the event of automatic deletion by the system,<br> the app will receive a rejection and have to resend the file.<br> If omitted, the value will be set to false.</td>
                  <td>N</td>
  			<td></td>
  			<td>SmartDeviceLink 2.0</td>
  		</tr>
  		<tr>
  			<td>systemFile</td>
  			<td>Boolean</td>
  			<td>Indicates if the file is meant to be passed thru core to elsewhere on the system. If set to TRUE, then the system will instead pass the data thru as it arrives to a predetermined area outside of core. If omitted, the value will be set to false.</td>
                  <td>N</td>
  			<td></td>
  			<td>SmartDeviceLink 2.3.2</td>
  		</tr>
  		<tr>
  			<td>offset</td>
  			<td>Float</td>
  			<td>Optional offset in bytes for resuming partial data chunks</td>
                  <td>N</td>
  			<td>Minvalue=0 <br>Maxvalue=100000000000</td>
  			<td>SmartDeviceLink 2.3.2</td>
  		</tr>
  		<tr>
  			<td>length</td>
  			<td>Float</td>
  			<td>Optional length in bytes for resuming partial data chunks. If offset is set to 0, then length is the total length of the file to be downloaded</td>
                  <td>N</td>
  			<td>Minvalue=0<br> Maxvalue=100000000000</td>
  			<td>SmartDeviceLink 2.3.2</td>
 		</tr>
   </table>

### Response ###

Response is sent, when the file data was copied (success case). Or when an error occurred.

### Non-default Result Codes: ###
```
- SUCCESS
- INVALID_DATA
- OUT_OF_MEMORY
- TOO_MANY_PENDING_REQUESTS
- APPLICATION_NOT_REGISTERED
- GENERIC_ERROR
- REJECTED
```
### Related Operations ###
```
- DeleteFile
- ListFiles
```
