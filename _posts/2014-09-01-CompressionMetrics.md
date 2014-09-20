---
layout: post
title: Compression - what is the right API to use?
category: Coding
tags: java Compression SNAPI LZ4 GZIP metrics ZLIB 
year: 2014
month: 09
day: 2
published: false
summary: Most of the pitfals of in-memory solutions is that they become expensive very fast as we try to store evrything in memory and keep 4 copies for redundency. Also modern applications dont behave properly while we try to use a huge heap size. All of the above would need to make a system elastically scalable and in the end unusable. 

image: clientserver.png
---

Compression is used by a system to support many of the below advantages

* Send large packets over network faster.
    * When the communication link is slower the data can be sent faster as we will be sending less number of bytes. 
    * Compression would be generally advantageous if request or response size is more than 1000 bytes, to reduce time spent on the network. 
* When we are constrained by space data can be compressed and stored.
* compression would also help while storing to a slow media.
	* We typicaly use SSD or HDD for storing data for long term storage which are 72 to 150+ times slower than temporary storage RAM. This is the reason why most high transactional systems are more memory bound.

Trying to compress and uncompress large number of bytes is counter productive so we would try to find a high bar and stick with that.

Now for some metrics on the different code that can be used for compression and de-compression using different API. In the below examples we will be only looking at the fastest compression/de-compression with the different API available with their speed and size of compression.

GZip code -

```
    public static byte[] serializeGZip(ByteBuffer buffer) throws IOException
    {
        ByteArrayOutputStream baos=new ByteArrayOutputStream();
        GZIPOutputStream gzipOs=new GZIPOutputStream(baos);

        try
        {
            byte[] bytes=new byte[4];
            ByteBuffer intBuffer=ByteBuffer.wrap(bytes);
            intBuffer.putInt(buffer.position());
            gzipOs.write(intBuffer.array());
            gzipOs.write(buffer.array(),0,buffer.position());
            gzipOs.finish();
            return baos.toByteArray();
        }
        finally
        {
            baos.close();
            gzipOs.close();
        }
    }

    public static byte[] deSerializeGZip(byte[] bytes) throws IOException
    {
        ByteArrayInputStream bais=new ByteArrayInputStream(bytes);
        GZIPInputStream gzipIs=new GZIPInputStream(bais);

        try {
            byte[] intBytes = new byte[4];
            gzipIs.read(intBytes);
            ByteBuffer intBuffer=ByteBuffer.wrap(intBytes);
            int length = intBuffer.getInt();

            byte[] unCompressed = new byte[length];
            gzipIs.read(unCompressed);

            return unCompressed;
        }
        finally
        {
            bais.close();
            gzipIs.close();
        }
    }
```

Snappy code -

```
    public static byte[] serializeSnappy(byte[] buffer,int position) throws IOException
    {
            return Snappy.rawCompress(buffer, position);
    }

    public static byte[] deSerializeSnappy(byte[] data) throws IOException
    {
            return Snappy.uncompress(data);
    }
```

LZF - ning compress code - 

```
    public static ByteBuffer serializelzf(ByteBuffer unCompressedBuffer) throws IOException
    {
        return ByteBuffer.wrap(LZFEncoder.encode(unCompressedBuffer.array()));
    }

    public static ByteBuffer deSerializelzf(ByteBuffer compressedBuffer) throws IOException
    {
        byte[] unCompressed=LZFDecoder.decode(compressedBuffer.array());
        return ByteBuffer.wrap(unCompressed);
    }
```

Deflator/Inflator code - 
```
    final static Deflater d = new Deflater(Deflater.BEST_SPEED);
    final static Inflater inf = new Inflater();
     public static ByteBuffer serializeZLib(ByteBuffer compressedBuffer,ByteBuffer unCompressedBuffer) throws IOException
    {
        d.setInput(unCompressedBuffer.array(),0,unCompressedBuffer.position());
        d.finish();

        compressedBuffer=resetAllocation(compressedBuffer,unCompressedBuffer.position());

        int length=d.deflate(compressedBuffer.array());
        compressedBuffer.putInt(length,unCompressedBuffer.position());
        compressedBuffer.position(length+4);
        d.reset();
        return compressedBuffer;
    }

    public static ByteBuffer deSerializeZLib(ByteBuffer compressedBuffer,ByteBuffer unCompressedBuffer)
            throws IOException,DataFormatException
    {
        int uncompressedLength = getUncompressedLength(compressedBuffer);

        resetAllocation(unCompressedBuffer,uncompressedLength);
        inf.setInput(compressedBuffer.array(), 0, compressedBuffer.position() - 4);

        inf.inflate(unCompressedBuffer.array(), 0, uncompressedLength);
        unCompressedBuffer.position(uncompressedLength);
        inf.reset();
        return unCompressedBuffer;
    }


```

LZ4 compression
```
	final static 	LZ4Factory 				m_factory = LZ4Factory.nativeInstance();
	final static 	LZ4Compressor 			m_compressor = m_factory.fastCompressor();
	final static LZ4FastDecompressor m_decompressor = m_factory.fastDecompressor();
	
public static ByteBuffer serializelz4(ByteBuffer unCompressedBuffer,ByteBuffer compressedBuffer) throws IOException
    {
        compressedBuffer=resetAllocation(compressedBuffer,unCompressedBuffer.position());

        int length=m_compressor.compress(unCompressedBuffer.array(),0,unCompressedBuffer.position(),
                              compressedBuffer.array(),0,unCompressedBuffer.position());
        compressedBuffer.putInt(length,unCompressedBuffer.position());
        compressedBuffer.position(length + 4);

        byte[] bytes=new byte[compressedBuffer.position()];
        System.arraycopy(compressedBuffer.array(),0,bytes,0,compressedBuffer.position());

        ByteBuffer buffer=ByteBuffer.wrap(bytes);
        buffer.position(compressedBuffer.position());
        return buffer;
    }

    public static ByteBuffer deSerializelz4(ByteBuffer unCompressedBuffer,ByteBuffer compressedBuffer) throws IOException
    {
        int uncompressedLength = getUncompressedLength(compressedBuffer);

        resetAllocation(unCompressedBuffer,uncompressedLength);
        m_decompressor.decompress(compressedBuffer.array(), unCompressedBuffer.array(),uncompressedLength);

        unCompressedBuffer.position(uncompressedLength);
        return unCompressedBuffer;
    }
```

common code -
```
	private static ByteBuffer resetAllocation(ByteBuffer buffer,int length)
    {
        if(buffer==null || buffer.capacity()<length)
        {
            buffer = ByteBuffer.allocate(length);
        }
        buffer.clear();
        return buffer;
    }

    private static int getUncompressedLength(ByteBuffer compressedBuffer)
    {
        int pos=0;
        if(compressedBuffer.position()==0)
        {
            pos=compressedBuffer.capacity()-4;
        }
        else
        {
            pos = compressedBuffer.position()-4;
        }

        return compressedBuffer.getInt(pos);
    }
```
 
There are two flavors of famous compression algorithms

Huffman (Tree Based) - Snappy, Inflate/Defelate, GZip 
lempel-ziv compression (Dictionary based) - LZ4, LZF  
 
Let us see different performance and compression metrics of the different flavors listed above and their advantages
 
Below graph shows the size of compression for different compression algorithms. 

 ![compression size](/img/compresssize.png)

Below graph shows the time taken in under seconds for un-compressing bytes based on the size 

 ![un-compression size](/img/compressiontime.png)

Below graph shows the time taken in under seconds for compression based on the size

 ![compression size](/img/uncompresstime.png)
 

<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

<div class="row">	
    <div class="span9 columns">    
		<h2>Comments Section</h2>
	    <p>Feel free to comment on the post but keep it clean and on topic.</p>	
		<div id="fb-root"></div>
<script>(function(d, s, id) {
  var js, fjs = d.getElementsByTagName(s)[0];
  if (d.getElementById(id)) return;
  js = d.createElement(s); js.id = id;
  js.src = "//connect.facebook.net/en_US/sdk.js#xfbml=1&version=v2.0";
  fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));</script>
<div class="fb-comments" data-href="http://vallur.github.io{{ page.url }}" data-numposts="5" data-width="700" data-colorscheme="light"></div>
</div>

<!-- Twitter -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

<!-- Google + -->
<script type="text/javascript">
  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>
