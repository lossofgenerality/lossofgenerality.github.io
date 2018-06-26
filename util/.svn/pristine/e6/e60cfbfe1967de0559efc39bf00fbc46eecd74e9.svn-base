<?php
namespace GIFEndec\Tests;

use GIFEndec\Decoder;
use GIFEndec\MemoryStream;
use GIFEndec\Frame;

class DecoderTest extends TestCase
{
    public function testDecode()
    {
        for ($i = 1; $i <= 6; $i++) {
            $this->decode("test$i");
        }
    }

    protected function decode($name)
    {
        $action = "split";
        $animation = __DIR__."/gifs/$name.gif";
        $checksumPath = __DIR__."/gifs/$name.$action.json";
        $dir = __DIR__."/output/$action/$name";

        $frameCount = 0;
        $checksums = [];
        $hasChecksums = $this->loadChecksums($checksumPath, $checksums);
        $this->createDirOrClear($dir);


        $stream = new MemoryStream();
        $stream->loadFromFile($animation);
        $decoder = new Decoder($stream);
        $decoder->decode(function (Frame $frame, $index) use (&$checksums, $hasChecksums, &$frameCount, $name, $dir) {
            $frameCount++;
            $paddedIndex = str_pad($index, 3, '0', STR_PAD_LEFT);

            $frame->getStream()->copyContentsToFile("$dir/frame{$paddedIndex}.gif");

            $checksum = sha1($frame->getStream()->getContents());
            if ($hasChecksums) {
                $this->assertEquals($checksum, $checksums[$index]);
            }
            $checksums[$index] = $checksum;
        });

        file_put_contents("$dir/$name.$action.json", json_encode($checksums, JSON_PRETTY_PRINT));
    }
}
