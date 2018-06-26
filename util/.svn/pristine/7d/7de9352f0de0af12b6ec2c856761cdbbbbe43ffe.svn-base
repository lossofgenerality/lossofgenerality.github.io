<?php

require __DIR__ . '/vendor/autoload.php';
/*
 * read all *.json files and merge
 * look through merged file and remove quizzes with bad image references
 * make backup of existing module.json
 */

if ($argc != 2) {
    \CCN\Util::error("Usage: validate <module dir>");
    die;
}

$rootdir = realpath($argv[1]);
\CCN\Util::error("$rootdir");

if (!file_exists($rootdir) || !is_dir($rootdir)) {
    \CCN\Util::error("Could not find the directory '$rootdir'");
    die;
}

$json = array();

foreach (glob("$rootdir" . DIRECTORY_SEPARATOR . "*.json") as $file) {
    \CCN\Util::error("merging $file");
    $json = \CCN\Util::mergejson($json, $file);
    if (basename($file) !== 'module.json') {
        \CCN\Util::error("renaming $file");
        rename($file, $file . '.bak');
    }
}

$module = CCN\Util::process_json($json, $rootdir);

\CCN\Util::make_backups();

file_put_contents($rootdir . DIRECTORY_SEPARATOR . "module.json", json_encode($module));

\CCN\Util::error(count($module) . " questions");
