
 # 7-zip Delphi API

This API use the 7-zip dll (7z.dll) to read and write all 7-zip supported archive formats.

- Autor: Henri Gourvest <hgourvest@progdigy.com>  
- Licence: MPL1.1  
- Date: 15/04/2009  
- Version: 1.1

## Reading archive:

### Extract to path:

```pascal
 with CreateInArchive(CLSID_CFormatZip) do 
 begin
      OpenFile('c:\test.zip');   
      ExtractTo('c:\test'); 
 end;
```

### Get file list:

```pascal
 with CreateInArchive(CLSID_CFormat7z) do 
 begin
      OpenFile('c:\test.7z');   
      for i := 0 to NumberOfItems - 1 do    
          if not ItemIsFolder[i] then writeln(ItemPath[i]); 
 end;
```

### Extract to stream

```pascal
 with CreateInArchive(CLSID_CFormat7z) do 
 begin   
      OpenFile('c:\test.7z');   
      for i := 0 to NumberOfItems - 1 do     
          if not ItemIsFolder[i] then ExtractItem(i, stream, false); 
 end;
```

### Extract "n" Items

```pascal
function GetStreamCallBack (     sender    : Pointer; 
                                 index     : Cardinal;  
                             var outStream : ISequentialOutStream) : HRESULT; stdcall;
begin  
     case index of ...    
          outStream := T7zStream.Create(aStream, soReference);  
     Result := S_OK;
end;


procedure TMainForm.ExtractClick(Sender: TObject);
const   
     items : array[0..2] of Cardinal = (0,1,2);
begin  
     with CreateInArchive(CLSID_CFormat7z) do  
     begin
          OpenFile('c:\test.7z');   // items must be sorted by index!
          ExtractItems(@items, Length(items), false, nil, GetStreamCallBack);
     end;
end;
```

### Open stream

```pascal
 with CreateInArchive(CLSID_CFormatZip) do begin   OpenStream(T7zStream.Create(TFileStream.Create('c:\test.zip', fmOpenRead), soOwned));   OpenStream(aStream, soReference);   ... end;
```

### Progress bar

```pascal
 function ProgressCallback(sender: Pointer; total: boolean; value: int64): HRESULT; stdcall; begin   if total then     Mainform.ProgressBar.Max := value else     Mainform.ProgressBar.Position := value;   Result := S_OK; end; procedure TMainForm.ExtractClick(Sender: TObject); begin   with CreateInArchive(CLSID_CFormatZip) do   begin     OpenFile('c:\test.zip');     SetProgressCallback(nil, ProgressCallback);     ...   end; end;
```

### Password

```pascal
 function PasswordCallback(sender: Pointer; var password: WideString): HRESULT; stdcall; begin
```

## Writing archive

```pascal
 procedure TMainForm.ExtractAllClick(Sender: TObject); var   Arch: I7zOutArchive; begin   Arch := CreateOutArchive(CLSID_CFormat7z);   // add a file   Arch.AddFile('c:\test.bin', 'folder\test.bin');   // add files using willcards and recursive search   Arch.AddFiles('c:\test', 'folder', '*.pas;*.dfm', true);   // add a stream   Arch.AddStream(aStream, soReference, faArchive, CurrentFileTime, CurrentFileTime, 'folder\test.bin', false, false);   // compression level   SetCompressionLevel(Arch, 5);   // compression method if <> LZMA   SevenZipSetCompressionMethod(Arch, m7BZip2);   // add a progress bar ...   Arch.SetProgressCallback(...);   // set a password if necessary   Arch.SetPassword('password');   // Save to file   Arch.SaveToFile('c:\test.zip');   // or a stream   Arch.SaveToStream(aStream); end;
```

