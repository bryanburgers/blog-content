
[somethinrg about secrets in state files]
[something about read-rights to state files, like an account that has read access to all S3 objects in an account]

## 1a. Create a PGP Key

The first thing you'll need is a PGP Key. PGP does public/private encryption, which means that you can give the public part to Terraform, store it in a public place, leave it in your state file and your git repo, etc. But only people with the private key can decrypt what is encrypted with the public key. 

    (Create key)
    ((Options))

You will then want to export the public/private key pair and store it somewhere safe.

    (Export key)

## 1b. Import a PGP Key

If you or someone else already followed Step 1a, you may need to import an existing key pair and use that instead.

Find the location of the existing key and run 

    (Import code)

## 2. Export the public key

The public key is what we'll put in our script file. Remember that this key should be safe to share as long as we keep the private key safe.

A major caveat here is that we previously used `--armor` to get an “armored file”. Terraform does not expect the key to be armored, so we need to omit `--armor` from this command.

    (Don't use --armor)

## 3. Write a Terraform script

Now that we have the key created, we can write the Terraform script that will create the secret. This can be (examples). I needed to encrypt access keys, so we'll use that as an example.

    aws_access_key {
    }

    output {
    }

## 4. Run the terraform script

    terraform plan -out planfile
    # verify we aren't taking down everything
    terraform apply

In the output, you should see a really long string of arbitrary characters. This is your encrypted value.

## 5. Check over your shoulder

Hackers are _everywhere_.

## 6. Decrypt the secret

When we're convinced hackers aren't right behind us, decrypt the secret by showing the secret on stdout, base64 decoding it, and running it through gpg.

    terraform output secret | \
      base64 --decode | \
      gpg decrypt -

The output will be the secret decrypted value that only you – and the hacker looking over your shoulder, you forgot to check, didn't you? – can see.

---

[some summary]
