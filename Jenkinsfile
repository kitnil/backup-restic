def exclude = resticExclude users: ["root", "oleg"], dirs: ["/var/lib/znc/moddata"]

def source = ["/home/oleg", "/etc", "/root"].join(" ")

pipeline {
    agent { label "master" }
    triggers {
        cron("H 20 * * *")
    }
    environment {
        RESTIC_PASSWORD = credentials("RESTIC_PASSWORD")
        RESTIC = "/home/oleg/.guix-profile/bin/restic"
    }
    stages {
        stage("Backup") {
            parallel {
                stage("Backup workstation") {
                    agent { label "workstation" }
                    steps {
                        sh (["sudo", "--login", "--preserve-env=RESTIC_PASSWORD",
                             RESTIC, "--repo", "sftp:backup.guix.duckdns.org:/srv/backup/majordomo",
                             exclude, "backup", source].join(" "))
                    }
                }
                stage("Backup spb") {
                    agent { label "spb" }
                    steps {
                        sh (["sudo", "--login", "--preserve-env=RESTIC_PASSWORD",
                             RESTIC, "--repo", "sftp:backup.guix.duckdns.org:/srv/backup/spb",
                             exclude, "backup", "/home/oleg", "/root", "/etc"].join(" "))
                    }
                }
                stage("Backup oracle") {
                    agent { label "oracle" }
                    steps {
                        sh (["sudo", "--preserve-env=RESTIC_PASSWORD",
                             "docker", "run",
                             "--security-opt", "label:disable",
                             "--rm",
                             "--name", "restic",
                             "--env", "RESTIC_PASSWORD=${RESTIC_PASSWORD}",
                             "--volume", "/home/oleg:/home/oleg",
                             "--volume", "/root:/root",
                             "--volume", "/opt/restic/rootfs/etc/passwd:/etc/passwd",
                             "--volume", "/var/lib/znc:/var/lib/znc",
                             "--volume", "/var/lib/bitlbee:/var/lib/bitlbee",
                             "localhost:5000/restic:latest",
                             "restic", "--repo", "sftp:backup.guix.duckdns.org:/srv/backup/oracle",
                             exclude, "backup", "/home/oleg", "/root", "/var/lib/znc", "/var/lib/bitlbee"].join(" "))
                    }
                }
            }
        }
        stage("Backup master") {
            agent { label "master" }
            steps {
                sh (["sudo", "--login", "--preserve-env=RESTIC_PASSWORD",
                     RESTIC, "--repo", "/srv/backup/guixsd",
                     exclude, "backup", source].join(" "))
            }
        }
    }
    post {
        always {
            sendNotifications currentBuild.result
        }
    }
}
