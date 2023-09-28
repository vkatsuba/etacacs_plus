# etacacs_plus

A simple TACACS+ server.

TACACS+ is described in RFC 8907 and is as a general Authentication,
Authorization, and Accounting (AAA) protocol (similar to Radius).

`etacacs_plus` is a simple implementation of a TACACS+ server and 
is primarily intended for testing of TACACS+ enabled applications.


## Build

    $ rebar3 compile
    
## Run

    $ rebar3 shell
    
Or by first building a release:

    # Build release
    $ rebar3 release
    
    # Run start script
    $ ./_build/default/rel/etacacs_plus/bin/etacacs_plus
    
    # Run start script with interative shell
    $ ./_build/default/rel/etacacs_plus/bin/etacacs_plus console
    
## Configuration

Configuration of IP/Port, the secret TACACS+ key and the user DB config file
is done in the `config/etacacs_plus.config` file.

    # Example of etacacs_plus.config content:
    [{etacacs_plus,
       [{key, "tacacs123"},
        {listen_ip, {0,0,0,0}},
        {port, 5049},
        {db_conf_file, "config/db.conf"}
       ]
     }
    ].

User data is configured in the `db.conf` file. The User/Password is
used for Authentiation and the User/Service is used for Authorization.

    # Example of db.conf content:
    {user, tacadmin,                           % the User
     [{login, {cleartext, "tacadmin"}},        % the user Password
      {service, nso,                           % for Authorization
       [{groups, [admin, netadmin, private]},  % returned data at success
        {uid, 1000},
        {gid, 100},
        {home, "/tmp"}
       ]
      },
      {member, [netadmin]}                     % not used
     ]
    }.

## Example usage

Using the TACACS+ Python client in: https://github.com/ansible/tacacs_plus

    # Authenticate
    $ tacacs_client -v -H 127.0.0.1 -p 5049 -k tacacs123 \
                    -u tacadmin authenticate 
    password for tacadmin: 
    status: PASS

    
    # Authorize the use of service: nso
    $ tacacs_client -v -H 127.0.0.1 -p 5049 -k tacacs123 \
                    -u tacadmin authorize  -c service=nso 
    status: PASS
    av-pairs:
      groups=admin netadmin private
      uid=1000
      gid=100
      home/tmp
      

    # Authorize the use of (the unknown) service: hello
    $ tacacs_client -v -H 127.0.0.1 -p 5049 -k tacacs123 \
                    -u tacadmin authorize  -c service=hello
    status: FAIL


