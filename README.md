use std::fmt;
use std::process::Command;

enum Environment {
    Dev,
    Prod,
}

impl fmt::Display for Environment {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            Environment::Dev => write!(f, "dev"),
            Environment::Prod => write!(f, "prod"),
        }
    }
}

struct App {
    name: String,
    environments: Vec<Environment>,
}

fn build_and_push(app: App) -> () {
    for environment in app.environments {
        println!(
            "Attempting to build the images: {}:{}",
            app.name,
            environment.to_string()
        );

        let mut build_command = Command::new("docker");

        let image_name =
            "adamcooper1386/ketosistant-".to_owned() + &app.name + ":" + &environment.to_string();
        let mut dockerfile_name = "Dockerfile";
        if let Environment::Dev = environment {
            dockerfile_name = "Dockerfile.dev";
        }
        let application_path = "/Users/adam/git/ketosistant/".to_owned() + &app.name;
        let docker_file_path =
            "/Users/adam/git/ketosistant/".to_owned() + &app.name + "/" + dockerfile_name;

        build_command
            .arg("build")
            .arg("--tag")
            .arg(image_name.clone())
            .arg("-f")
            .arg(docker_file_path)
            .arg(application_path);

        if let Ok(mut child) = build_command.spawn() {
            child.wait().expect("build command not running");
            println!("child process of build_command finished");

            let mut push_command = Command::new("docker");
            push_command.arg("push").arg(image_name);

            if let Ok(mut child) = push_command.spawn() {
                child.wait().expect("push command not running");
                println!("child process of push command finished")
            } else {
                println!("docker push command did not start")
            }
        } else {
            println!("docker build didn't start");
        }
    }
}

fn main() {
    let apps = [
        App {
            name: "web".to_string(),
            environments: vec![Environment::Dev, Environment::Prod],
        },
        App {
            name: "pocketbase".to_string(),
            environments: vec![Environment::Prod],
        },
    ];

    for app in apps {
        build_and_push(app);
    }
}
