# HashiCorp `vagrant` mock of **`vault`** with an AppRole.

This repo is a WIP exploring the example detailed on [the learn guide of Vault ApprRole](https://learn.hashicorp.com/tutorials/vault/approle).


## Usage & Workflow

```bash
vagrant up --provider virtualbox ;
# // … output of provisioning steps.

vagrant ssh hsm1-vault1 ;  # // SSH to Leader Vault node.
  # ………
#vagrant@hsm1-vault1:~$ \

# // 1. enabling:
vault auth enable approle ;
# // 2. create role in AppRole to be used:
vault write auth/approle/role/mf-workordersecrets-1 policies=default secret_id_ttl="16m" token_max_ttl="5400s" ;
# // 3. Verify role properties just created:
vault read auth/approle/role/mf-workordersecrets-1 ;
# // 4. Use created role to generate secret-id:
VROLE_ID=$(vault read auth/approle/role/mf-workordersecrets-1/role-id -format=json | jq -r '.data.role_id') ;  # // role_id for later ref
vault write -f auth/approle/role/mf-workordersecrets-1/secret-id -format=json > vapp_sid1.json ;  # // accessor_id for later ref
# // 5. Note current expiry of secret:
vault write -field=expiration_time /auth/approle/role/mf-workordersecrets-1/secret-id-accessor/lookup secret_id_accessor=$(jq -r '.data.secret_id_accessor' vapp_sid1.json) ;
# // 6. Destroy former secret_id:
vault write -f /auth/approle/role/mf-workordersecrets-1/secret-id/destroy role_id=$VROLE_ID secret_id=$(jq -r '.data.secret_id' vapp_sid1.json) ;
# // 7. Re-generate a new secret-id:
vault write -f auth/approle/role/mf-workordersecrets-1/secret-id -format=json > vapp_sid2.json ;
# // ...OPTIONAL - List current accessors: vault list /auth/approle/role/mf-workordersecrets-1/secret-id ;
# // 8. Note expiry time from secret ID accessor
vault write -field=expiration_time /auth/approle/role/mf-workordersecrets-1/secret-id-accessor/lookup secret_id_accessor=$(jq -r '.data.secret_id_accessor' vapp_sid2.json) ;

# // exit ...

# // ---------------------------------------------------------------------------
# when completely done:
vagrant destroy -f hsm1-vault1 ;
vagrant box remove -f debian/buster64 --provider virtualbox ; # ... delete box images
```

------
