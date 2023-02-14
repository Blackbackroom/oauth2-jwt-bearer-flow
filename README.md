OAuth 2.0 JWT Bearer Flow for Server-to-Server Integration

# Saleforce documentation
https://help.salesforce.com/s/articleView?id=sf.remoteaccess_oauth_jwt_flow.htm&type=5

# Generate private key
# The genrsa command generates an RSA private key, which essentially
#      involves the generation of two prime numbers.  When generating the key,
#      various symbols will be output to indicate the progress of the
#      generation.  A ‘.’ represents each number which has passed an initial
#      sieve test; ‘+’ means a number has passed a single round of the Miller-
#      Rabin primality test; ‘*’ means the number has failed primality testing
#      and needs to be generated afresh.  A newline means that the number has
#      passed all the prime tests (the actual number depends on the key size).
# 
#  -aes128 | -aes192 | -aes256 | -camellia128 | -camellia192 | -camellia256
#              | -des | -des3 | -idea
#              Encrypt the private key with the AES, CAMELLIA, DES, triple DES
#              or the IDEA ciphers, respectively, before outputting it.  If none
#              of these options are specified, no encryption is used.  If
#              encryption is used, a pass phrase is prompted for, if it is not
#              supplied via the -passout option.
# 
#  -passout arg
#              The output file password source.
# 
#  -out file
#              The output file to write to, or standard output if not specified.
# 
# numbits
#              The size of the private key to generate in bits.  This must be
#              the last option specified.  The default is 2048.
openssl genrsa -des3 -passout pass:SomePassword -out server.pass.key 2048

# Process RSA key
# The rsa command processes RSA keys.  They can be converted between
#      various forms and their components printed out.  rsa uses the traditional
#      SSLeay compatible format for private key encryption: newer applications
#      should use the more secure PKCS#8 format using the pkcs8 utility.
# 
#  -passin arg
#              The key password source.
# 
# -in file
#              The input file to read from, or standard input if not specified.
#              If the key is encrypted, a pass phrase will be prompted for.
# 
# -out file
#              The output file to write to, or standard output if not specified.
openssl rsa -passin pass:SomePassword -in  server.pass.key -out server.pem

# Create certificate request
#  The req command primarily creates and processes certificate requests in
#      PKCS#10 format.  It can additionally create self-signed certificates, for
#      use as root CAs, for example.
# 
# -new    Generate a new certificate request.  The user is prompted for the
#              relevant field values.  The actual fields prompted for and their
#              maximum and minimum sizes are specified in the configuration file
#              and any requested extensions.
# 
#              If the -key option is not used, it will generate a new RSA
#              private key using information specified in the configuration
#              file.
# 
# -key keyfile
#              The file to read the private key from.  It also accepts PKCS#8
#              format private keys for PEM format files.
# 
# -out file
#              The output file to write to, or standard output if not specified.
openssl req -new -key server.key -out server.csr

# Generate certificate from request. It should be used in Connected App
# The x509 command is a multi-purpose certificate utility.  It can be used
#      to display certificate information, convert certificates to various
#      forms, sign certificate requests like a "mini CA", or edit certificate
#      trust settings.
# 
# -days arg
#            The number of days to make a certificate valid for.  The default is
#            30 days.
# 
# -req  Expect a certificate request on input instead of a certificate.
# 
# -out file
#            The output file to write to, or standard output if none is
#            specified.
# 
# -in file
#            The input file to read from, or standard input if not specified.
# 
# -signkey file
#            Self-sign file using the supplied private key.
# 
#            If the input file is a certificate, it sets the issuer name to the
#            subject name (i.e. makes it self-signed), changes the public key to
#            the supplied value, and changes the start and end dates.  The start
#            date is set to the current time and the end date is set to a value
#            determined by the -days option.  Any certificate extensions are
#            retained unless the -clrext option is supplied.
# 
#            If the input is a certificate request, a self-signed certificate is
#            created using the supplied private key using the subject name in
#            the request.
# 
#  -md5 | -sha1
#            The digest to use.  This affects any signing or display option that
#            uses a message digest, such as the -fingerprint, -signkey, and -CA
#            options.  If not specified, MD5 is used.  SHA1 is always used with
#            DSA keys.
openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt

# https://www.willhaley.com/blog/generate-jwt-with-bash/
# Generate signature
# https://jwt.io/
# Get timestamp (Time skew 3 mins max)
# https://www.unixtimestamp.com/

# JWT example
# {
# 	“alg”: “RS256”,
# 	“typ”: “JWT”
# }
# {
# 	“iss”: “client_id”,
# 	“sub”: “username”,
# 	“aud”: “https://login.salesforce.com”,
# 	“exp”: “unixtimestamp”
# }

# Create keystore to upload into SF instance
# The pkcs12 command allows PKCS#12 files (sometimes referred to as PFX
#      files) to be created and parsed.  By default, a PKCS#12 file is parsed; a
#      PKCS#12 file can be created by using the -export option.
# 
# -export
#            Create a PKCS#12 file (rather than parsing one).
# 
# -in file
#            The input file to read from, or standard input if not specified.
#            The order doesn't matter but one private key and its corresponding
#            certificate should be present.  If additional certificates are
#            present, they will also be included in the PKCS#12 file.
# 
#      -inkey file
#            File to read a private key from.  If not present, a private key
#            must be present in the input file.
# 
# -out file
#            The output file to write to, or standard output if not specified.
openssl pkcs12 -export -in server.crt -inkey server.pem -out testkeystore.p12

# Convert pkcs12 into jks with alias 1
keytool pkcs12  -importkeystore testkeystore.p12 -srcstoretype pkcs12 -destkeystore servercert.jks  -deststoretype JKS

# Change keystore's alias
keytool -keystore ./servercert.jks -changealias -alias 1 -destalias salesforcetest
