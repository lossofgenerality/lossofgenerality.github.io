<?php

namespace CCN;

class Util {

    static function error($msg) {
        fwrite(STDERR, "$msg\n");
    }

    static function make_backups() {
        //backups
        @rename("$rootdir" . DIRECTORY_SEPARATOR . "module.json.4", "$rootdir" . DIRECTORY_SEPARATOR . "module.json.5");
        @rename("$rootdir" . DIRECTORY_SEPARATOR . "module.json.3", "$rootdir" . DIRECTORY_SEPARATOR . "module.json.4");
        @rename("$rootdir" . DIRECTORY_SEPARATOR . "module.json.2", "$rootdir" . DIRECTORY_SEPARATOR . "module.json.3");
        @rename("$rootdir" . DIRECTORY_SEPARATOR . "module.json.1", "$rootdir" . DIRECTORY_SEPARATOR . "module.json.2");
        @rename("$rootdir" . DIRECTORY_SEPARATOR . "module.json", "$rootdir" . DIRECTORY_SEPARATOR . "module.json.1");
    }

    static function fixGif($image, $rootdir) {
        $tp = tempnam($rootdir, 'AGIF');
        $fcnt = Util::fixAnimatedGif($image, basename($tp), $rootdir);
        if ($fcnt > 1) {
            Util::error("Fixed $image ($fcnt frames)");
            rename($tp, "$rootdir/$image");
            chmod("$rootdir/$image", 0644);
        } else {
            unlink($tp);
        }
    }

    static function convert_file($image, $rootdir) {

        if (!is_object($image)) {
            $file = $rootdir . DIRECTORY_SEPARATOR . $image;
            $ftype = strtolower(pathinfo($file, PATHINFO_EXTENSION));

            switch ($ftype) {
                case 'eps':
                    $svgfile = substr($image, 0, -strlen($ftype)) . 'SVG';
                    if (!is_file($svgfile)) {
                        Util::error("converting $image to $svgfile");
                        exec("inkscape $file -l " . $rootdir . DIRECTORY_SEPARATOR . $svgfile);
                    }
                    $image = $svgfile;
                    break;
            }
        }

        return $image;
    }

    static function validate_file($file, $rootdir) {
        $optimize = true;

        if (is_object($file)) {
            if (property_exists($file, 'optimize')) {
                $optimize = $file->optimize;
            }
            $file = $file->image;
        }

        if (!Util::image_exists($file, $rootdir)) {
            $missing = true;
        } else {

            $isvalid = true;
            $ftype = strtolower(pathinfo($file, PATHINFO_EXTENSION));

            switch ($ftype) {
                case 'gif':
                    $isvalid = Util::isValidAnimatedGif($file, $rootdir);
                    if ($isvalid) {
                        if ($optimize) {
                            Util::fixGif($file, $rootdir);
                        } else {
                            Util::error("don't optimize $file");
                        }
                    } else {
                        Util::error("bad GIF $file");
                    }
                    break;

                default:
                    break;
            }


            $missing = !$isvalid;
        }

        return !$missing;
    }

    static function mergejson($json, $file) {
        $content = file_get_contents($file);
        if (false === $content) {
            Util::error("There was a problem reading $file");
            die;
        }

        $newjson = json_decode($content);
        if (null == $newjson) {
            Util::error("There was an error parsing the file");
            die;
        }

        return array_merge($json, $newjson);
    }

    static function image_exists($image, $path) {
        if (!file_exists($path . DIRECTORY_SEPARATOR . $image)) {
            Util::error($path . DIRECTORY_SEPARATOR . $image . " does not exist");
            return false;
        }
        return true;
    }

    static function eps2svg($image, $path) {
        
    }

    //https://github.com/stil/gif-endec
    static function isValidAnimatedGif($image, $path) {
        $badframes = 0;
        $file = $path . DIRECTORY_SEPARATOR . $image;

        if (strtolower(pathinfo($file, PATHINFO_EXTENSION)) == 'gif') {
            $gifStream = new \GIFEndec\IO\FileStream($file);
            $gifDecoder = new \GIFEndec\Decoder($gifStream);
            $gifDecoder->decode(function (\GIFEndec\Events\FrameDecodedEvent $event) use (&$badframes) {
                if ($event->decodedFrame->getSize()->getWidth() == 0 || $event->decodedFrame->getSize()->getHeight() == 0) {
                    $badframes++;
                }
            });
            if ($badframes > 0) {
                Util::error($path . DIRECTORY_SEPARATOR . $image . " has $badframes bad frames");
            }
        }

        return ($badframes == 0);
    }

    static function fixAnimatedGif($source, $target, $path) {
        $framecount = 0;
        if (strtolower(pathinfo($source, PATHINFO_EXTENSION)) == 'gif') {
            $gifStream = new \GIFEndec\IO\FileStream("$path/$source");
            $gifDecoder = new \GIFEndec\Decoder($gifStream);
            $filehash = '';
            $newgif = new \GIFEndec\Encoder();
            $gifDecoder->decode(function (\GIFEndec\Events\FrameDecodedEvent $event) use (&$filehash, &$newgif, &$framecount) {
                if ($event->decodedFrame->getSize()->getWidth() != 0 && $event->decodedFrame->getSize()->getHeight() != 0) {

                    $path = tempnam(sys_get_temp_dir(), 'AGIF');
                    $event->decodedFrame->getStream()->copyContentsToFile($path);
                    $h = sha1_file($path);
                    if ($h != $filehash) {
                        $filehash = $h;
                        $event->decodedFrame->setDuration(30);
                        $newgif->addFrame($event->decodedFrame);
                        $framecount++;
                    }
                    unlink($path);
                }
            });
            $newgif->getStream()->copyContentsToFile("$path/$target");
        }
        return $framecount;
    }

    static function process_json($json, $rootdir) {
        $module = array();
        foreach ($json as &$quiz) {
            $quizzes = array();
            foreach ($quiz as &$question) {
                $missing = false;
                foreach (array('question', 'anime', 'choices', 'tutorial', 'tutorial_anime', 'pdf', 'cdf', 'hint', 'example', 'example_anime') as $element) {
                    if (property_exists($question, $element)) {
                        $e = $question->$element;
                        if (is_array($e)) {
                            $choices = $e;
                            foreach ($choices as $key => &$file) {
                                $fmissing = !Util::validate_file($file, $rootdir);
                                if (!$fmissing) {
                                    $question->$element[$key] = Util::convert_file($file, $rootdir);
                                }
                                $missing = $missing || $fmissing;
                            }
                        } else {
                            $fmissing = !Util::validate_file($e, $rootdir);
                            if (!$fmissing) {
                                $question->$element = Util::convert_file($e, $rootdir);
                            }
                            $missing = $missing || $fmissing;
                        }
                    }
                }
                if (!$missing) {
                    $quizzes[] = $question;
                }
            }
            if (count($quizzes) == count($quiz)) {
                $module[] = $quizzes;
            } else {
                Util::error("module had bad question(s), not including");
            }
        }
        return $module;
    }

}
