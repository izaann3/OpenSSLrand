# Generación y Gestión de Claves para Cifrado y Firma Digital en un Sistema de Mensajería Segura

**Objetivo**: Implementar un sistema de mensajería segura utilizando claves criptográficas generadas con openssl rand. Aprenderemos a aplicar conceptos de criptografía simétrica y asimétrica, generación de claves seguras, y autenticación de mensajes.

---

### Escenario

Somos parte del  equipo de seguridad de una empresa que necesita proteger su sistema de mensajería para evitar que los mensajes sean interceptados, alterados o enviados por usuarios no autorizados. Para ello, implementaremos un sistema de cifrado y firma digital basado en claves generadas con OpenSSL.

---

### Instrucciones

#### Parte 1: Generación de Claves de Usuario y Configuración Inicial

1. **Generar una clave simétrica de 64 bytes para el cifrado de mensajes**:
   - Usa `openssl rand` para crear una clave de 64 bytes en formato hexadecimal y guarda esta clave en un archivo llamado `clave_simetrica.hex`.

   ```bash
   openssl rand -hex 64 -out clave_simetrica.hex
### Generar un par de claves asimétricas (pública y privada) para cada usuario

- Cada usuario (Bonilla y Ruben) debe tener su propio par de claves para firmar y verificar los mensajes.
- Genera el par de claves RSA de 2048 bits para Bonilla y guárdalos en los archivos `bonilla_privada.pem` y `bonilla_publica.pem`.

    ```bash
    openssl genpkey -algorithm RSA -out bonilla_privada.pem -pkeyopt rsa_keygen_bits:2048
    openssl rsa -pubout -in bonilla_privada.pem -out bonilla_publica.pem
    ```

- Repite los pasos anteriores para Ruben, guardando sus claves en `ruben_privada.pem` y `ruben_publica.pem`.

---

### Parte 2: Envío de Mensajes Cifrados y Firmados

#### Cifrar un mensaje

- Crea un mensaje en un archivo llamado `mensaje.txt` con el texto “Mensaje para Rubencito con Samba”.
- Utiliza la clave simétrica generada en la Parte 1 para cifrar el mensaje usando AES-256-CBC, y guarda el mensaje cifrado en `mensaje_cifrado.enc`.

    ```bash
    openssl enc -aes-256-cbc -in mensaje.txt -out mensaje_cifrado.enc -pass file:clave_simetrica.hex
    ```

#### Firmar el mensaje cifrado

- Usa la clave privada de Bonilla (`bonilla_privada.pem`) para crear una firma digital del mensaje cifrado.
- Almacena la firma en un archivo llamado `firma_mensaje.bin`.

    ```bash
    openssl dgst -sha256 -sign bonilla_privada.pem -out firma_mensaje.bin mensaje_cifrado.enc
    ```

#### Compartir la clave simétrica de forma segura

- Cifra el archivo `clave_simetrica.hex` con la clave pública de Ruben para que solo él pueda descifrarla. Guarda el resultado en `clave_simetrica_cifrada.enc`.

    ```bash
    openssl rsautl -encrypt -inkey ruben_publica.pem -pubin -in clave_simetrica.hex -out clave_simetrica_cifrada.enc
    ```

---

### Parte 3: Recepción y Verificación de Mensajes

#### Descifrar la clave simétrica

- Ruben recibe `clave_simetrica_cifrada.enc` y usa su clave privada (`ruben_privada.pem`) para descifrar la clave simétrica y guardarla en `clave_simetrica_ruben.hex`.

    ```bash
    openssl rsautl -decrypt -inkey ruben_privada.pem -in clave_simetrica_cifrada.enc -out clave_simetrica_ruben.hex
    ```

#### Descifrar el mensaje

- Con la clave simétrica descifrada, Ruben puede descifrar el mensaje original usando AES-256-CBC, y guarda el mensaje en `mensaje_descifrado.txt`.

    ```bash
    openssl enc -d -aes-256-cbc -in mensaje_cifrado.enc -out mensaje_descifrado.txt -pass file:clave_simetrica_ruben.hex
    ```

#### Verificar la firma del mensaje

- Ruben verifica la autenticidad del mensaje cifrado utilizando la clave pública de Bonilla (`bonilla_publica.pem`).

    ```bash
    openssl dgst -sha256 -verify bonilla_publica.pem -signature firma_mensaje.bin mensaje_cifrado.enc
    ```

- Si la firma es válida, Ruben sabe que el mensaje fue enviado por Bonilla y no ha sido alterado.

ssl_protocols TLSv1.2 TLSv1.3; 
ssl_prefer_server_ciphers on; 
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_session_cache shared:SSL:10m; 
ssl_session_timeout 10m; 
ssl_stapling on; 
ssl_stapling_verify on;
