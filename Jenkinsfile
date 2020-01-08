def exclude = [
    "/root/.cache",
    "/home/oleg/.cache",
    "/home/oleg/.guix-profile",
    "/home/oleg/.nix-profile",
    "/home/oleg/Downloads",
    "/home/oleg/GNS3",
    "/home/oleg/Videos",
    "/home/oleg/archive",
    "/home/oleg/majordomo",
    "/home/oleg/src",
    "/home/oleg/vm",
].collect{"--exclude $it"}.join(" ")

def source = ["/home/oleg", "/etc", "/root"].join(" ")

pipeline {
    agent { label "master" }
    environment {
        RESTIC_PASSWORD = credentials("RESTIC_PASSWORD")
        RESTIC = "/home/oleg/.guix-profile/bin/restic"
    }
    stages {
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
                         exclude, "--exclude", "/root/.cache",
                         "backup", "/home/oleg", "/root", "/etc"].join(" "))
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
                         "--interactive",
                         "--tty",
                         "--env", "RESTIC_PASSWORD=${RESTIC_PASSWORD}",
                         "--volume", "/home/oleg:/home/oleg",
                         "--volume", "/root:/root",
                         "--volume", "/opt/restic/rootfs/etc/passwd:/etc/passwd",
                         "--volume", "/var/lib/znc:/var/lib/znc",
                         "--volume", "/var/lib/bitlbee:/var/lib/bitlbee",
                         "localhost:5000/restic:latest",
                         "restic", "--repo", "sftp:backup.guix.duckdns.org:/srv/backup/oracle",
                         "--exclude", "/home/oleg/.cache",
                         "--exclude", "/root/.cache",
                         "--exclude", "/var/lib/znc/moddata",
                         "backup", "/home/oleg", "/root", "/var/lib/znc", "/var/lib/bitlbee"].join(" "))
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
