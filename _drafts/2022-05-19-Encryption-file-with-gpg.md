
GPG File Encryption with passphrase

* Environment
  * Ubuntu 20.04
  * gpg (GnuPG) 2.2.19
  * libgcrypt 1.8.5

* Gpg 다운로드
  * Package Manager
    * sudo apt install gpg
  * Source codes
    * https://git.gnupg.org/cgi-bin/gitweb.cgi?p=gnupg.git

* Encryption 
  GUI가 없는 shell에서 한다고 가정. (특히, 서버)

  * Encryption
    * gpg -c --pinentry-mode loopback --passphrase "<password>" -o <destination> <file>
    * -c: encryption
    * -o: destination file / 설정하지 않으면 stdout
    * --pinentry-mode: 정확한 것은 모르지만, loopback으로 설정하면 CUI 모드에서도 gpg 사용 가능
    * --armor: encryption 파일이 ascii로 생성(hex)
      * 권장하지 않음

  * Decryption
    * gpg -d --pinentry-mode loopback --passphrase "<password>" -o <destination> <file>
    * -d: decryption
    * -o: destination file / 설정하지 않으면 stdout
    * --pinentry-mode: encryption과 동일
    

