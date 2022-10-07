# Notes 

notes 配置

## Caddyfile
```Caddyfile
@notes {
    not path /notes
    not path /notes/
    not path /notes/readme
    not path /notes/readme/
    path /notes/*
    path_regexp ^/notes/(.*)$
}

route @notes {
    uri strip_prefix /notes
    file_server {
        root /srv/public/notes
    }
}

route /hook/notes {
    basicauth {
        hook xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    }
    exec bash /exec/hook-notes.sh {
        timeout 120s
    }
}
```