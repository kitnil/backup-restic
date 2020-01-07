def exclude = ["/home/oleg/.cache",
               "/home/oleg/Downloads",
               "/home/oleg/Downloads",
               "/home/oleg/GNS3/images",
               "/home/oleg/GNS3/projects",
               "/home/oleg/archive/src",
               "/home/oleg/src/replicant",
               "/home/oleg/vm",
               "/home/oleg/Videos",
               "/home/oleg/.guix-profile",
               "/home/oleg/.nix-profile",
               "/home/oleg/src/guix",
               "/home/oleg/src/guix-wip-forge",
               "/home/oleg/src/guix-wip-origin",
               "/home/oleg/src/guix-wip-channels",
               "/home/oleg/src/macos-simple-kvm"].collect{"--exclude $it"}.join(" ")

def source = ["/home/oleg", "/etc"].join(" ")

pipeline {
    agent { label "master" }
    environment {
        RESTIC_PASSWORD = credentials("RESTIC_PASSWORD")
        RESTIC = "/home/oleg/.guix-profile/bin/restic"
    }
    stages {
        stage("Backup master") {
            agent { label "master" }
            steps {
                sh (["sudo", "--login", "--preserve-env=RESTIC_PASSWORD",
                     RESTIC, "--repo", "/srv/backup/guixsd",
                     exclude, "backup", source].join(" "))
            }
        }
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
    }
    post {
        always {
            sendNotifications currentBuild.result
        }
    }
}
